---
title: "Parent-Child Communication Patterns in F# Elmish"
date: 2025-09-13
draft: false
tags: ['F#', 'Elmish', 'Feliz', 'React', 'Functional Programming', 'Architecture Patterns']
---

Often when building larger Elmish applications with Feliz, there comes a time when you need child components to talk back to their parents. This is closely followed by head-scratching as I try to remember what the best pattern is and inevitably ending up with tightly coupled components that are a pain to test and reuse.

The below guide should set you straight until the next time you forget which approach to use.

## TL;DR

**Simple callback** - for short-lived components (dialogs, popups)  
**State lifting** - when parent needs to validate or coordinate children  
**Hybrid approach** - for complex components that need autonomy but still communicate events

Skip to the [complete example](#complete-example) if you want to see the hybrid approach in action.

## The Problem

I was building a file uploader component recently. It needed to handle its own complex state (upload progress, validation errors, retry logic) but also tell the parent when files were uploaded successfully or when things went wrong.

Sound familiar? You want the component to be self-contained but not completely isolated.

## Three Patterns That Actually Work

### 1. Simple Callbacks

When your child component's job is basically "do this thing and tell me when you're done":

```fsharp
// Parent passes a simple callback
let onSave = SaveFormData >> dispatch

// Child uses it and disappears
Button.create [
    Button.onClick (fun _ -> onSave formData)
    Button.text "Save & Close"
]
```

Works great for dialogs, confirmations, anything that has a short lifecycle.

### 2. State Lifting

When the parent needs to be involved in what the child is doing:

```fsharp
type ParentMsg =
    | FormFieldMsg of FormField.Msg
    | ValidateForm

let update msg model =
    match msg with
    | FormFieldMsg fieldMsg ->
        let fieldModel, fieldCmd = FormField.update fieldMsg model.FormField
        { model with FormField = fieldModel }, 
        Cmd.map FormFieldMsg fieldCmd
```

This is your bread-and-butter pattern when children need coordination or the parent has to validate things.

### 3. The Hybrid Approach

This is where it gets interesting. Your child manages its own complexity but emits specific events that the parent might care about:

```fsharp
// Child defines what events it can emit
type ExternalMsg =
    | FileUploaded of fileName: string * fileSize: int64
    | UploadFailed of fileName: string * error: string
    | AllUploadsComplete

// Parent gets a callback for these events
type Props = {
    MaxFileSize: int64
    OnExternalMsg: ExternalMsg -> unit
}
```

The child handles all its internal complexity (progress tracking, errors, retries) but tells the parent about the outcomes that matter.

## Making It Work with React.useElmish

The cleanest way I've found is to call the external callback directly in your update function:

```fsharp
[<ReactComponent>]
let FileUploaderComponent (props: Props) =
    let model, dispatch = React.useElmish(
        (fun () -> init props.MaxFileSize, Cmd.none),
        (fun msg model ->
            match msg with
            | UploadComplete fileName ->
                // Handle your internal state
                let newModel = { model with UploadProgress = Map.add fileName 100 model.UploadProgress }
                
                // Tell the parent what happened
                props.OnExternalMsg (FileUploaded (fileName, file.size))
                
                newModel, Cmd.none
            // ... other cases
        ),
        [| props.OnExternalMsg |]
    )
```

The parent then decides what to do with those events:

```fsharp
let handleFileUploaderMsg = function
    | FileUploader.FileUploaded (name, size) ->
        AddUploadedFile (name, size) |> dispatch
    | FileUploader.UploadFailed (name, error) ->
        ShowNotification $"Upload failed: {error}" |> dispatch
    | FileUploader.AllUploadsComplete ->
        ShowNotification "All done!" |> dispatch
```

## When to Use What

**Simple callbacks** when:

- Component disappears after doing its job
- No validation or coordination needed
- One-way communication is enough

**State lifting** when:

- Parent needs to validate child state
- Multiple children need to work together
- Parent behaviour changes based on child state

**Hybrid approach** when:

- Child has complex internal logic
- Parent only cares about specific outcomes
- You want the component to be reusable

## Complete Example

A file uploader that validates files, tracks upload progress, handles errors, but only tells the parent about the important stuff:

```fsharp
module FileUploader

open Elmish
open Feliz
open Browser.Types

type Model = {
    SelectedFiles: File list
    UploadProgress: Map<string, int>
    IsUploading: bool
    Errors: Map<string, string>
    MaxFileSize: int64
    AllowedExtensions: string list
}

type Msg =
    | FilesSelected of File list
    | StartUpload
    | UploadComplete of fileName: string
    | UploadError of fileName: string * error: string
    | RemoveFile of fileName: string

type ExternalMsg =
    | FileUploaded of fileName: string * fileSize: int64
    | UploadFailed of fileName: string * error: string
    | ValidationError of fileName: string * reason: string
    | AllUploadsComplete

type Props = {
    MaxFileSize: int64
    AllowedExtensions: string list
    OnExternalMsg: ExternalMsg -> unit
}

let validateFile (file: File) model =
    let extension = file.name.Split('.') |> Array.last |> fun ext -> $".{ext.ToLower()}"
    let sizeMB = file.size / (1024L * 1024L)
    
    if file.size > model.MaxFileSize then
        Some $"File too large: {sizeMB}MB"
    elif not (List.contains extension model.AllowedExtensions) then
        Some $"File type not allowed: {extension}"
    else
        None

let uploadFileCmd fileName =
    let uploadTask = async {
        do! Async.Sleep 2000 // Simulate upload time
        if System.Random().Next(0, 4) = 0 then
            return UploadError (fileName, "Network timeout")
        else
            return UploadComplete fileName
    }
    Cmd.OfAsync.perform uploadTask () id

[<ReactComponent>]
let FileUploaderComponent (props: Props) =
    let model, dispatch = React.useElmish(
        (fun () -> {
            SelectedFiles = []
            UploadProgress = Map.empty
            IsUploading = false
            Errors = Map.empty
            MaxFileSize = props.MaxFileSize
            AllowedExtensions = props.AllowedExtensions
        }, Cmd.none),
        (fun msg model ->
            match msg with
            | FilesSelected files ->
                let validFiles = files |> List.filter (fun f -> validateFile f model |> Option.isNone)
                let invalidFiles = files |> List.filter (fun f -> validateFile f model |> Option.isSome)
                
                // Tell parent about validation failures
                invalidFiles |> List.iter (fun f ->
                    match validateFile f model with
                    | Some error -> props.OnExternalMsg (ValidationError (f.name, error))
                    | None -> ()
                )
                
                { model with SelectedFiles = validFiles }, Cmd.none
                
            | StartUpload when not model.IsUploading ->
                let uploadCmds = 
                    model.SelectedFiles 
                    |> List.map (fun f -> uploadFileCmd f.name)
                    |> Cmd.batch
                
                { model with IsUploading = true }, uploadCmds
                
            | UploadComplete fileName ->
                let file = model.SelectedFiles |> List.find (fun f -> f.name = fileName)
                
                // Tell parent about successful upload
                props.OnExternalMsg (FileUploaded (fileName, file.size))
                
                // Check if all files are done
                let allComplete = 
                    model.SelectedFiles 
                    |> List.forall (fun f -> 
                        f.name = fileName || Map.containsKey f.name model.UploadProgress)
                
                if allComplete then
                    props.OnExternalMsg AllUploadsComplete
                
                { model with UploadProgress = Map.add fileName 100 model.UploadProgress }, Cmd.none
                
            | UploadError (fileName, error) ->
                // Tell parent about failure
                props.OnExternalMsg (UploadFailed (fileName, error))
                { model with 
                    Errors = Map.add fileName error model.Errors
                    IsUploading = false }, Cmd.none
                
            | RemoveFile fileName ->
                let newFiles = model.SelectedFiles |> List.filter (fun f -> f.name <> fileName)
                { model with SelectedFiles = newFiles }, Cmd.none
        ),
        [| props.OnExternalMsg |]
    )

    Html.div [
        prop.children [
            Html.input [
                prop.type' "file"
                prop.multiple true
                prop.onChange (fun (files: File list) -> 
                    FilesSelected files |> dispatch)
            ]
            
            Html.div [
                prop.children [
                    for file in model.SelectedFiles do
                        Html.div [
                            prop.children [
                                Html.span [prop.text file.name]
                                Html.button [
                                    prop.text "Remove"
                                    prop.onClick (fun _ -> RemoveFile file.name |> dispatch)
                                ]
                                
                                // Show progress if uploading
                                match Map.tryFind file.name model.UploadProgress with
                                | Some progress -> Html.div [prop.text $"Progress: {progress}%%"]
                                | None -> Html.none
                                
                                // Show errors
                                match Map.tryFind file.name model.Errors with
                                | Some error -> Html.div [prop.text error; prop.style [style.color color.red]]
                                | None -> Html.none
                            ]
                        ]
                ]
            ]
            
            Html.button [
                prop.text (if model.IsUploading then "Uploading..." else "Start Upload")
                prop.disabled (model.IsUploading || List.isEmpty model.SelectedFiles)
                prop.onClick (fun _ -> StartUpload |> dispatch)
            ]
        ]
    ]
```

## Using It in a Parent Component

```fsharp
[<ReactComponent>]
let ParentComponent () =
    let model, dispatch = React.useElmish(init, update, [| |])
    
    let handleFileUploaderMsg = function
        | FileUploader.FileUploaded (name, size) ->
            AddUploadedFile (name, size) |> dispatch
        | FileUploader.UploadFailed (name, error) ->
            ShowError $"Failed to upload {name}: {error}" |> dispatch
        | FileUploader.AllUploadsComplete ->
            ShowSuccess "All files uploaded!" |> dispatch
        | FileUploader.ValidationError (name, reason) ->
            ShowWarning $"Validation failed for {name}: {reason}" |> dispatch

    Html.div [
        prop.children [
            FileUploader.FileUploaderComponent {
                MaxFileSize = 10L * 1024L * 1024L // 10MB
                AllowedExtensions = [".pdf"; ".jpg"; ".png"]
                OnExternalMsg = handleFileUploaderMsg
            }
            // ... rest of your UI
        ]
    ]
```

## Things to Watch Out For

**Don't over-engineer** - if a simple callback works, use it. The hybrid approach adds complexity that's only worthwhile when you actually need it.

**Remember your dependencies** - include the callback in your `React.useElmish` dependency array: `[| props.OnExternalMsg |]`

**Keep external messages meaningful** - instead of `StateChanged`, use descriptive names like `ValidationFailed` or `UploadCompleted`.

The important thing is matching the communication pattern to what you're actually trying to achieve. Start simple and only add complexity when you need it.

---

A blog seems easier than remembering things.
