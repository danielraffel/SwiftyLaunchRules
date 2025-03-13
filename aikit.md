## AIKit

AIKit integrates AI capabilities into your app through a collection of mini-applications powered by OpenAI. When implementing features with AIKit, follow these guidelines to ensure proper integration of AI capabilities while maintaining performance and user experience.

### Setup and Configuration

1. **OpenAI API Setup**:
   - Obtain an OpenAI API key with appropriate permissions.
   - Store the API key securely in the BackendKit `config.ts` file.
   - Never include API keys in client-side code to prevent exposure.

2. **BackendKit Integration**:
   - AIKit requires BackendKit for secure API key management and server-side processing.
   - Configure BackendKit following the setup instructions before using AIKit features.
   - Ensure the OpenAI API key is properly set in the `config.ts` file.

3. **Resource Management**:
   - Be aware that the Whisper model (for speech-to-text) is bundled with your app (approximately 77MB).
   - Consider the impact on app size when including the AIKit module.
   - When using OpenAI services, be mindful of API rate limits and costs.

### Implementation Guidelines

1. **AI Vision Features**:
   - Use the server-side `accessGPTVision()` function for image analysis:
   ```typescript
   // Server-side code
   const analysisResult = await AI.accessGPTVision(
     imageBase64,
     "What objects are in this image?"
   );
   ```
   - Convert images to Base64 format before sending to the server:
   ```swift
   // Client-side code
   let base64Image = yourUIImage.base64EncodedString()
   ```
   - Implement proper error handling for image processing failures.

2. **AI Chat/Text Processing**:
   - Use the server-side `accessGPTChat()` function for text generation:
   ```typescript
   // Server-side code
   const response = await AI.accessGPTChat({
     text: userPrompt,
     previousChatMessages: chatHistory
   });
   ```
   - Maintain chat context by properly formatting and sending message history:
   ```typescript
   // Format for chat history
   const chatHistory: GPTChatMessage[] = [
     { role: "user", content: "Hello" },
     { role: "assistant", content: "Hi there! How can I help you?" }
   ];
   ```
   - Be careful with prompt construction to ensure accurate AI responses.

3. **Text-to-Speech Integration**:
   - Use the server-side `convertTextToMp3Base64Audio()` function to generate audio:
   ```typescript
   // Server-side code
   const audioBuffer = await AI.convertTextToMp3Base64Audio(textToSpeak);
   ```
   - Convert the Buffer to Base64 string when returning to the client:
   ```typescript
   return {
     text: responseText,
     audio: audioBuffer.toString("base64")
   };
   ```
   - On the client side, convert Base64 back to audio data for playback.

4. **Speech Recognition**:
   - Use the built-in Whisper model for on-device speech recognition:
   ```swift
   // Client-side recording
   voiceRecordingVM.startRecording()
   
   // Process transcription when available
   func handleTranscription(_ text: String) {
     // Send to backend or process locally
   }
   ```
   - Be aware of microphone permission requirements and handle authorization properly.

### User Experience and Performance

1. **Loading States**:
   - Implement clear loading indicators for AI operations:
   ```swift
   @State private var isProcessing = false
   
   Button("Process") {
     isProcessing = true
     Task {
       // AI processing
       await performAIOperation()
       isProcessing = false
     }
   }
   .disabled(isProcessing)
   .overlay(isProcessing ? ProgressView() : nil)
   ```
   - Provide appropriate feedback during longer operations like image analysis.

2. **Error Handling**:
   - Implement comprehensive error handling for AI operations:
   ```swift
   do {
     let result = try await aiOperation()
     // Handle success
   } catch AIError.rateLimitExceeded {
     // Handle rate limiting
   } catch AIError.modelUnavailable {
     // Handle service issues
   } catch {
     // Handle general errors
   }
   ```
   - Provide graceful fallbacks when AI services are unavailable.

3. **Battery and Network Optimization**:
   - Be mindful of battery usage when implementing AI features:
   ```swift
   // Check network conditions before heavy operations
   if NetworkMonitor.shared.isConnectedToWiFi {
     // Perform high-bandwidth operation
   } else {
     // Offer to delay or use lighter alternative
   }
   ```
   - Consider offline fallbacks for critical features.

### Integration with Other Modules

1. **With AuthKit and DBKit**:
   - AIKit integrates with AuthKit to store chat history:
   ```swift
   // Ensure user is authenticated before saving chat history
   guard let userId = Auth.auth().currentUser?.uid else {
     // Handle unauthenticated user
     return
   }
   
   // Save chat history to Firestore
   db.collection("users").document(userId).collection("chats").add(chatData)
   ```
   - Use DBKit to persist and retrieve AI conversation history.

2. **With AnalyticsKit**:
   - AIKit automatically tracks events when AnalyticsKit is present:
   ```swift
   // Events are tracked automatically, but you can add custom tracking
   Analytics.capture(
     .info,
     id: "ai_image_processed",
     longDescription: "User processed image with AI vision",
     source: .aikit
   )
   ```
   - Monitor AI feature usage to identify optimization opportunities.

### Mini-App Customization

1. **Vision AI (Bunny L1)**:
   - Customize the camera UI and image processing workflow:
   ```swift
   // Extend the existing camera view
   struct CustomVisionExampleView: View {
     @StateObject private var viewModel = VisionExampleViewModel()
     
     var body: some View {
       VStack {
         CameraView(viewModel: viewModel.cameraViewModel)
         // Custom UI components
       }
     }
   }
   ```
   - Implement image analysis with custom prompts:
   ```swift
   // Send image to backend with custom prompt
   let result = await db.processImageWithAI(
     jpegImageBase64: imageBase64,
     processingCommand: "Identify and count all animals in this image",
     readResultOutLoud: true
   )
   ```
   - Support for camera feed, photo library selection, and voice question input.

2. **AI Voice Translator**:
   - Adapt the translation prompt for specific use cases:
   ```swift
   // Customize translation style or domain
   let customPrompt = "Translate the following text into \(targetLanguage) using technical terminology appropriate for a medical context: \(sourceText)"
   ```
   - Add support for multiple languages through a language selector:
   ```swift
   Menu {
     Picker("Will translate to:", selection: $vm.selectedOutputLanguage) {
       ForEach(languages, id: \.self) {
         Text($0)
       }
     }
   } label: {
     HStack {
       Text("Will translate to \(vm.selectedOutputLanguage)")
     }
   }
   ```
   - Use on-device speech recognition with the Whisper model for transcription.

3. **AI ChatBot**:
   - Customize the chat interface and conversation style:
   ```swift
   // Initialize ChatBot with custom system prompt
   let customSystemPrompt = "You are a customer service assistant for a fitness app. Help users with workout plans and nutrition advice."
   
   // Use when starting new chat
   viewModel.startNewChat(withSystemPrompt: customSystemPrompt)
   ```
   - Implement persistence of chat history using DBKit:
   ```swift
   // Fetch existing chat history
   let history = await db.fetchAIChatHistory()
   
   // Send new message
   let messages = await db.sendNewAIChatMessage(text: userInput)
   ```
   - Support for deleting chat history:
   ```swift
   let success = await db.deleteAIChatHistory()
   ```

### Backend Implementation

1. **API Integration**:
   - Use the provided AIKit functions in your BackendKit endpoints:
   ```typescript
   // Import AIKit functions
   import * as AI from "./AIKit/AI";
   
   // Define endpoint
   export const analyzeImageContents = onCall(async (request) => {
     const imageBase64 = request.data?.imageBase64 as string | null;
     const processingCommand = request.data?.processingCommand as string | null;
     
     // Call AI function
     const result = await AI.accessGPTVision(imageBase64, processingCommand);
     
     return { message: result };
   });
   ```
   - Create type definitions for consistent data structures:
   ```typescript
   type ImageAnalysisResultType = {
     message: string;
     audio: string | null;
   };
   ```

2. **Audio Processing**:
   - Generate audio responses when requested:
   ```typescript
   if (readOutLoud) {
     const audioBuffer = await AI.convertTextToMp3Base64Audio(textResponse);
     return {
       message: textResponse,
       audio: audioBuffer.toString("base64")
     };
   }
   ```

3. **Client-Server Communication**:
   - Define consistent interfaces for server communication in your Swift code:
   ```swift
   public struct ImageAnalysisResultType: Codable {
     public let message: String
     public let audio: String?
     
     public init(from rawData: [String: Any]) throws {
       // Parse from raw data
     }
   }
   
   public func processImageWithAI(
     jpegImageBase64: String,
     processingCommand: String,
     readResultOutLoud: Bool
   ) async -> ImageAnalysisResultType? {
     // Implementation
   }
   ```

### Security and Privacy

1. **User Data Protection**:
   - Implement proper data handling for user conversations:
   ```swift
   // Clear sensitive data when no longer needed
   func clearConversationHistory() {
     chatHistory = []
     // If saved to DB
     if let userId = Auth.auth().currentUser?.uid {
       db.collection("users").document(userId).collection("chats").document(chatId).delete()
     }
   }
   ```
   - Be transparent about data usage in your privacy policy.

2. **Content Moderation**:
   - Implement content filters for user inputs:
   ```swift
   func validateUserInput(_ input: String) -> Bool {
     // Check for inappropriate content or sensitive information
     return !containsProfanity(input) && !containsPersonalIdentifiers(input)
   }
   ```
   - Use OpenAI's moderation endpoints for additional protection.

3. **Rate Limiting and Cost Control**:
   - Implement rate limiting to prevent abuse and control costs:
   ```typescript
   // Server-side rate limiting
   const userRequests = await getRateLimit(userId);
   if (userRequests > MAX_REQUESTS_PER_DAY) {
     throw new Error("Daily limit exceeded");
   }
   ```
   - Consider implementing tiered access based on user subscription level.

### Permission Handling

1. **Microphone Access**:
   - Use the `askUserFor` helper for microphone permissions:
   ```swift
   askUserFor(.microphoneAccess) {
     // Permission granted, start recording
     voiceRecordingViewModel.shouldStartRecording()
   } onDismiss: {
     // Permission denied
     showInAppNotification("Microphone access is required for voice recording")
   }
   ```

2. **Camera Access**:
   - Use the `requireCapabilityPermission` modifier for camera access:
   ```swift
   CameraView(session: cameraViewModel.session)
     .requireCapabilityPermission(
       of: .cameraAccess,
       onSuccess: {
         vm.gotCameraPermissions(cameraVM: cameraViewModel)
       },
       onCancel: {
         // Handle cancellation
       }
     )
   ```

3. **Photo Library Access**:
   - Use built-in SwiftUI components for photo access:
   ```swift
   PhotosPicker(
     selection: $vm.selectedImagePickerItem,
     matching: .images,
     preferredItemEncoding: .compatible
   ) {
     Image(systemName: "photo.stack")
   }
   ```

### Implementation Examples

1. **Processing Images with AI Vision**:
   ```swift
   // Take a picture with the camera
   Button {
     cameraViewModel.captureImage()
   } label: {
     Image(systemName: "camera.circle.fill")
   }
   
   // When image is captured
   .onChange(of: cameraViewModel.capturedImage) { newImage in
     if let image = newImage, let base64Image = image.jpegData(compressionQuality: 0.8)?.base64EncodedString() {
       Task {
         if let result = await db.processImageWithAI(
           jpegImageBase64: base64Image,
           processingCommand: "Describe what you see in this image",
           readResultOutLoud: true
         ) {
           // Handle the result
           if let audio = result.audio {
             audioPlayer.playAudio(base64Source: audio)
           } else {
             showDescription(result.message)
           }
         }
       }
     }
   }
   ```

2. **Voice Recording and Translation**:
   ```swift
   // Voice recording button
   Button {
     // Button released
     viewModel.releasedTranslateButton(voiceRecordingVM: voiceRecordingViewModel)
   } label: {
     Image(systemName: "mic.circle.fill")
   }
   .simultaneousGesture(
     LongPressGesture(minimumDuration: 0.2).onEnded { _ in
       askUserFor(.microphoneAccess) {
         voiceRecordingViewModel.shouldStartRecording()
       }
     }
   )
   
   // When transcription is ready
   .onChange(of: voiceRecordingViewModel.currentAudioTranscription) { transcription in
     if let text = transcription {
       Task {
         if let result = await db.processTextWithAI(
           text: "Translate the following text into French: \(text)",
           readResultOutLoud: true
         ) {
           // Handle translation result
         }
       }
     }
   }
   ```

3. **AI Chat Implementation**:
   ```swift
   // Load chat history
   Task {
     let history = await db.fetchAIChatHistory()
     messages = history
   }
   
   // Send new message
   func sendMessage(_ text: String) {
     Task {
       let newMessages = await db.sendNewAIChatMessage(text: text)
       messages.append(contentsOf: newMessages)
     }
   }
   
   // Clear chat history
   func clearHistory() {
     Task {
       if await db.deleteAIChatHistory() {
         messages = []
       }
     }
   }
   ```
