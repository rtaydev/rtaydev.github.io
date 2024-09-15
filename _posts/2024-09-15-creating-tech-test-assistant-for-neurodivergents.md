---
layout: post
title: "Creating a Tech Test Assistant Flutter App for macOS Desktop"
date: 2024-09-15
categories: [Tech, Flutter, macOS]
tags: ["neurodivergence", "tech interviews", "Flutter", "macOS", "Google Vision", "OpenAI"]
excerpt: "Discover how to create a Flutter app for macOS that assists neurodivergent individuals during tech interviews by leveraging Google Vision for OCR and OpenAI for natural language processing."
author: "Russ Taylor"
image: /assets/gifs/nd-tech-assist.gif
---

## Introduction

Tech interviews are stressful for many developers, but for neurodivergent individuals, the artificial and high-pressure environment of a tech test can be overwhelming. Neurodivergence, which includes ADHD, autism, and dyslexia, often comes with difficulties in handling stress, processing information quickly, and maintaining focus. In an interview setting, these challenges can cause candidates to freeze or hyper-focus on the interviewer, ultimately impacting their performance in ways that do not reflect their technical abilities.

Interviews, especially tech tests, are often designed in a way that simulates "real" coding environments but are far from genuine. The combination of time limits, being observed, and solving unfamiliar problems can trigger intense anxiety for neurodivergent individuals, making it harder to showcase their true skills.

### Challenges in Tech Interviews

Freezing or Hyper-Focusing on the Interviewer
Neurodivergent individuals often experience "freezing" during interviews—where their mind goes blank, and they can’t proceed with the task, no matter how simple. This can happen when they focus too much on the interviewer instead of the problem at hand. The constant awareness of being observed can make it difficult to enter a "flow" state, a critical element of productive coding.

### Anxiety and Emulated Situations

Interviews are artificial by nature, and for neurodivergent individuals, they often heighten stress rather than simulate a real-world coding environment. Many struggle to adapt to the contrived conditions of tech tests, especially when interviewers expect immediate solutions without giving time for reflection.

### Overcoming Interview Anxiety

Preparation and using tools that reduce cognitive strain are essential strategies for neurodivergent candidates. By building tools that help automate or offload some of the mental load (e.g., quickly parsing problem statements or suggesting code patterns), candidates can focus more on their technical strengths and worry less about performance anxiety.

---

### Disclaimer: When and How to Use This Tool

If you're using this tool during a real tech interview because you are neurodivergent, it's always a good idea to inform the interviewer. Mention that you're using a tool to help manage the cognitive load during the interview, and request any reasonable accommodations you may need. Alternatively, this app is also an excellent way to practice coding challenges before an interview, helping you stay focused and organized.

The source code for this tool is available on GitHub: [GitHub Repository](https://github.com/your-repo/neuro-test-assistant).

---

## Creating a Tech Test Assistant Flutter App for macOS Desktop

This app will assist you during tech tests by utilizing Google Vision for OCR (Optical Character Recognition) and OpenAI for natural language processing to provide helpful responses based on captured text.

### Step 1: Set Up Your Environment

1. **Install Flutter**: Follow the [official Flutter installation guide](https://flutter.dev/docs/get-started/install).
2. **Enable macOS Desktop Support**:

   ```sh
   flutter config --enable-macos-desktop
   ```

### Step 2: Creating a New Flutter Project

1. **Create a New Project**:

   ```sh
   flutter create nd_test_assistant
   ```

2. **Navigate to the Project Directory**:

   ```sh
   cd nd_test_assistant
   ```

### Step 3: Adding Dependencies

In your `pubspec.yaml`, add the necessary dependencies for API communication and logging:

```yaml
dependencies:
flutter:
    sdk: flutter
logger: ^1.3.0
google_vision: ^2.0.0
flutter_dotenv: ^5.0.2
```

These dependencies serve specific purposes in the app. The `logger` package is used for logging and debugging messages, which is crucial when handling external API responses and internal state changes. The `google_vision` package allows us to perform OCR by communicating with Google’s Vision API, and `flutter_dotenv` is used to load API keys from a `.env` file for security purposes.

### Step 4: Setting Up Google Cloud and OpenAI APIs

#### Google Cloud Vision API

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project and enable the **Vision API**.
3. Generate an **API Key** under "Credentials".
4. Store the API key in an `.env` file:

   ```sh
   GOOGLE_API_KEY=your_google_api_key
   ```

#### OpenAI API

1. Sign up for [OpenAI](https://beta.openai.com/signup/).
2. Generate an **API Key** under the API section.
3. Store the key in the `.env` file:

   ```sh
   OPENAI_API_KEY=your_openai_api_key
   ```

Make sure the `.env` file is loaded in `pubspec.yaml`:

```yaml
assets:
  - .env
```

### Step 5: Flutter Code Overview

The core functionality of this app relies on two services: one for performing OCR via Google Vision, and the other for generating responses from OpenAI based on the extracted text. Let’s break down each part.

#### Main Entry Point (`main.dart`)

The `main.dart` file is the entry point for our Flutter app. It initializes the application by loading the necessary environment variables and setting up the user interface.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'package:logger/logger.dart';
import 'package:nd_test_assistant/screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await dotenv.load(fileName: ".env");

  runApp(MainApp());
}

class MainApp extends StatelessWidget {
  MainApp({super.key});
  final logger = Logger();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: HomeScreen(),
    );
  }
}
```

This code initializes the `dotenv` package to load environment variables from the `.env` file, ensuring that the sensitive API keys are not hardcoded into the app. The `MainApp` widget is a stateless widget that returns the primary UI, which is the `HomeScreen`. The `Logger` object is initialized here, allowing us to log important information for debugging purposes throughout the app.

#### OCR Service (`ocr_service.dart`)

The `OcrService` class is responsible for interacting with the Google Vision API. This service reads an image file, sends it to Google Vision for processing, and returns any text detected in the image.

```dart
import 'package:google_vision/google_vision.dart';
import 'package:logger/logger.dart';
import 'dart:io';
import 'package:flutter_dotenv/flutter_dotenv.dart';

class OcrService {
  static final logger = Logger(
    printer: PrettyPrinter(),
  );

  Future<String?> performOcrAndLog(String imagePath) async {
    try {
      final apiKey = dotenv.get('GOOGLE_API_KEY');
      final googleVision = GoogleVision().withApiKey(apiKey);
      logger.i('Performing OCR on image...');

      final buffer = await File(imagePath).readAsBytes();
      final byteBuffer = buffer.buffer;

      List<EntityAnnotation> ocrResponse = await googleVision.image
          .textDetection(JsonImage.fromBuffer(byteBuffer));

      final textAnnotations = ocrResponse.map((e) => e.description).firstWhere(
          (description) => description.isNotEmpty,
          orElse: () => '');

      logger.i('Extracted text: \\n$textAnnotations');
      return textAnnotations;
    } catch (e) {
      logger.e(e);
      return null;
    }
  }
}
```

This service begins by loading the Google Vision API key from the environment variables using `dotenv.get('GOOGLE_API_KEY')`. It then uses the `GoogleVision` instance to perform OCR on an image. The `performOcrAndLog` method first reads the image file from the specified `imagePath`, converting it into bytes that can be processed by the API.

The `textDetection` method returns a list of `EntityAnnotation` objects, which contain the detected text from the image. The code maps through the list and returns the first non-empty `description`, which represents the recognized text. This process is logged using the `Logger` for debugging.

#### OpenAI Service (`openai_service.dart`)

The `OpenaiService` class is responsible for sending the extracted text to OpenAI and receiving a response, which can be used to assist the developer during tech tests.

```dart
import 'dart:convert';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'package:http/http.dart' as http;

class OpenaiService {
  final String apiKey = dotenv.get('OPENAI_API_KEY');
  final String apiUrl = "https://api.openai.com/v1/chat/completions";

  OpenaiService();

  Stream<String> sendMessageStream(String message) async* {
    final request = http.Request('POST', Uri.parse(apiUrl));
    request.headers.addAll({
      'Content-Type': 'application/json',
      'Authorization': 'Bearer $apiKey',
    });

    request.body = jsonEncode({
      "model": "gpt-4",
      "messages": [
        {
          "role": "system",
          "content": "You are a senior mobile and web app engineer specializing in React and React Native..."
        },
        {"role": "user", "content": message}
      ],
      "max_tokens": 1500,
      "temperature": 0.7,
      "stream": true,
    });

    final httpClient = http.Client();
    final response = await httpClient.send(request);

    final stream = response.stream.transform(utf8.decoder).transform(const LineSplitter());

    await for (var line in stream) {
      if (line.startsWith('data: ')) {
        final jsonData = line.substring(6);
        if (jsonData.trim() == '[DONE]') {
          break;
        }
        if (jsonData.isNotEmpty) {
          final decodedJson = jsonDecode(jsonData);
          final deltaContent = decodedJson['choices'][0]['delta']['content'];
          if (deltaContent != null) {
            yield deltaContent;
          }
        }
      }
    }
  }
}
```

In this service, we interact with the OpenAI API by sending a POST request to the `/chat/completions` endpoint. This service takes the extracted text (from the OCR service) as input and sends it to OpenAI, which processes it using the GPT-4 model. The `sendMessageStream` method enables streaming of the response, allowing the UI to display incremental results as OpenAI generates them.

The `http.Request` object is created to make the API call. It includes the necessary headers for authentication, with the API key loaded from the environment variables. The request body contains the model type (GPT-4), the user prompt (extracted text), and parameters such as `max_tokens` and `temperature` to control the behavior of the AI response.

This method leverages a `Stream` to yield the generated response in chunks as they are received, making the application more interactive by not waiting for the entire response to be generated before displaying results.

---

### macOS Swift Code

In addition to the Flutter app, the macOS version of the tool requires native Swift code to handle platform-specific features, such as screenshot capture and window management. Let’s go over the key Swift code components used to manage these features.

#### AppDelegate.swift

The `AppDelegate.swift` file initializes the app and sets up the screenshot manager when the application launches.

```swift
import Cocoa
import FlutterMacOS

@main
class AppDelegate: FlutterAppDelegate {

  override func applicationShouldTerminateAfterLastWindowClosed(_ sender: NSApplication) -> Bool {
    return true
  }

  override func applicationDidFinishLaunching(_ notification: Notification) {
    ScreenshotManager.shared.setupScreenshotChannel(mainFlutterWindow: mainFlutterWindow)
    
    // Set the initial position and size of the window
    if let window = mainFlutterWindow, let screenSize = NSScreen.main?.frame.size {
      WindowManager.shared.setInitialWindowSizeAndPosition(window: window, screenSize: screenSize)
    }
  }
}
```

This code serves as the entry point for the macOS-specific functionality. It sets up the `ScreenshotManager` and configures the main app window. The `applicationDidFinishLaunching` method is crucial because it initializes the platform channel used to communicate between Flutter and native macOS code, enabling functionality such as screenshots and window manipulation.

#### WindowManager.swift

The `WindowManager` class manages the window sizing, positioning, and style for the macOS app.

```swift
import Cocoa
import FlutterMacOS

class WindowManager {
  static let shared = WindowManager()

  private init() {}

  func setInitialWindowSizeAndPosition(window: NSWindow?, screenSize: CGSize) {
    let windowSize = CGSize(width: 400, height: 300)
    let x = screenSize.width - windowSize.width
    let y = screenSize.height - windowSize.height
    window?.setFrame(NSRect(x: x, y: y, width: windowSize.width, height: windowSize.height), display: true)
    window?.styleMask.insert([.resizable, .titled])
  }

  func configureMainWindow(controller: FlutterViewController) {
    let window = controller.view.window
    window?.backgroundColor = NSColor.clear
    window?.isOpaque = false
    window?.hasShadow = false
    window?.styleMask.remove(.titled)
    window?.styleMask.remove(.resizable)
    window?.styleMask.remove(.closable)
    window?.ignoresMouseEvents = false
    window?.level = .floating
  }

  func setupOverlayWindow(screenSize: CGRect) {
    ScreenshotManager.shared.overlayWindow = NSWindow(contentRect: screenSize,
                                                      styleMask: [.borderless],
                                                      backing: .buffered,
                                                      defer: false)
    ScreenshotManager.shared.overlayWindow?.isOpaque = false
    ScreenshotManager.shared.overlayWindow?.backgroundColor = NSColor.clear
    ScreenshotManager.shared.overlayWindow?.ignoresMouseEvents = false
    ScreenshotManager.shared.overlayWindow?.level = .screenSaver
  }
}
```

The `WindowManager` class provides methods to configure the app’s window. It allows you to set the initial size and position of the window relative to the screen. It also sets up the overlay window, which is used during the screenshot selection process. This overlay is transparent, allowing users to visually select a region of the screen for capture.

The `configureMainWindow` method ensures that the main Flutter window is properly set up, removing unnecessary features like the title bar and making the window floating, which ensures it stays on top of other windows.

#### ScreenshotManager.swift

The `ScreenshotManager` class handles the logic for capturing screenshots, interacting with the macOS native screen capture APIs.

```swift
import Cocoa
import FlutterMacOS
import ScreenCaptureKit

class ScreenshotManager {
  static let shared = ScreenshotManager()
  var overlayWindow: NSWindow?

  private init() {}

  func setupScreenshotChannel(mainFlutterWindow: NSWindow?) {
    guard let flutterViewController = NSApplication.shared.windows.first(where: { $0.contentViewController is FlutterViewController })?.contentViewController as? FlutterViewController else {
      return
    }

    WindowManager.shared.configureMainWindow(controller: flutterViewController)
    setupMethodChannel(controller: flutterViewController)
  }

  private func setupMethodChannel(controller: FlutterViewController) {
    let screenshotChannel = FlutterMethodChannel(name: "com.ndtechassistant.screenshot", binaryMessenger: controller.engine.binaryMessenger)
    screenshotChannel.setMethodCallHandler { [weak self] (call: FlutterMethodCall, result: @escaping FlutterResult) in
      guard let self = self else { return }
      if call.method == "drawRectAndCapture" {
        self.showOverlayWindow(result: result)
      } else {
        result(FlutterMethodNotImplemented)
      }
    }
  }

  private func showOverlayWindow(result: @escaping FlutterResult) {
    guard let screenSize = NSScreen.main?.frame else {
      result(FlutterError(code: "UNAVAILABLE", message: "Screen size unavailable", details: nil))
      return
    }
    WindowManager.shared.setupOverlayWindow(screenSize: screenSize)
    
    let view = CaptureView(frame: screenSize, result: result)
    overlayWindow?.contentView = view
    overlayWindow?.makeKeyAndOrderFront(nil)
    
    if let mainFlutterWindow = NSApplication.shared.windows.first(where: { $0.contentViewController is FlutterViewController })?.contentViewController?.view.window {
      mainFlutterWindow.orderOut(nil)
    }
    
    NSApp.activate(ignoringOtherApps: true)
    
    view.onCaptureComplete = {
      if let mainFlutterWindow = NSApplication.shared.windows.first(where: { $0.contentViewController is FlutterViewController })?.contentViewController?.view.window {
        mainFlutterWindow.makeKeyAndOrderFront(nil)
      }
      self.overlayWindow?.orderOut(nil)
    }
  }
}
```

The `ScreenshotManager` uses macOS's native `ScreenCaptureKit` to capture screenshots. It first creates an overlay window that lets users select a portion of the screen. Once the selection is made, the screenshot is captured and returned to the Flutter app. This class also sets up the communication channel between Flutter and macOS through a `FlutterMethodChannel`, allowing Flutter to trigger screenshot functionality.

### GitHub Repository

The full source code for this project is available on GitHub: [GitHub Repository](https://github.com/rtaydev/nd-tech-assistant).
