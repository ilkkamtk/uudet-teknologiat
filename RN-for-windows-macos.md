# React Native for Windows + macOS

## Introduction

[React Native for Windows + macOS ](https://microsoft.github.io/react-native-windows/)is a project that adds support for the Windows 10 SDK and macOS 10.12+ to React Native. The project is a fork of the main React Native repository, and it is developed and maintained by Microsoft.

## Why extend React Native to desktop?

- **Code reuse**: React Native allows you to write your app once and run it on multiple platforms. By extending React Native to Windows and macOS, you can target desktop platforms with the same codebase.
   - This is the end goal. Still a new project, so not all features are supported yet.
- **Native performance**: React Native for Windows + macOS uses native components and APIs to provide a native experience on Windows and macOS.
   - This means that RN components have native performance but business logic is still in JS.
   - Performance of fully native apps is still better overall.
- **Leverage existing skills**: If you are already familiar with React Native, you can use the same skills to build desktop apps.
- **Leverage existing code**: If you have an existing React Native app, you can extend it to Windows and macOS without rewriting the entire app.

## Use cases

- Business productivity apps
  - e.g. email clients, chat apps, project management tools
  - commonly used in both mobile and desktop
- Cross-platform tools with desktop needs
   - e.g. developer tools
- Apps that need to integrate with the OS
   - e.g. file system access, system notifications, system tray

## Advantages

- Shared codebase across mobile and desktop. 
- Access to native desktop APIs and UI components. 
- Microsoft’s active support and tooling (for Windows). 
- Leverage React ecosystem with additional desktop support.

## Limitations

- More platform-specific coding than mobile. 
- Smaller community and ecosystem for Windows/macOS compared to iOS/Android. 
- Limited third-party library support for desktop features.


## Platform-specific features

### Windows

- App lifecycle and windowing model
   - Windows apps have different lifecycle events and windowing model compared to mobile.
- [WinUI](https://microsoft.github.io/react-native-windows/docs/0.67/winui3)
  - a native UI framework for Windows apps.

### macOS

- Integration with [macOS AppKit](https://developer.apple.com/documentation/appkit)
   - macOS apps have different UI components and APIs compared to Windows.
- Using macOS-specific native modules and components
   - e.g. system menu, touch bar, macOS notifications.
- Handling macOS-specific gestures and window behavior
   - e.g. swipe gestures, window resizing.

## Getting started

- Check system requirements and install prerequisites
   - [Windows](https://microsoft.github.io/react-native-windows/docs/rnw-dependencies)
   - [macOS](https://microsoft.github.io/react-native-windows/docs/rnm-dependencies)

### Installation
- [Windows](https://microsoft.github.io/react-native-windows/docs/getting-started)
- [macOS](https://microsoft.github.io/react-native-windows/docs/rnm-getting-started)

## Exercise: Create a React Native For Windows + macOS App

---

#### Objective:
Develop a Markdown editor with **React Native application** for **Windows** or **macOS**. Features:
1. Markdown editor with preview.
   - `<TextInput>` for editing Markdown text.
   - E.g. [react-native-markdown-display](https://www.npmjs.com/package/react-native-markdown-display) for preview.
2. Save and load Markdown files.
   - Files need to be saved/loaded from the local file system.
   - Library for file system access: react-native-fs
      - [Windows version](https://github.com/birdofpreyru/react-native-fs)
      - [macOS version](https://github.com/itinance/react-native-fs)

---


