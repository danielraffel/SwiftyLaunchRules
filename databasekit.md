## DatabaseKit

DatabaseKit (DBKit) wraps Firebase Firestore SDK (or Supabase for Supabase projects) to provide seamless database interaction in your app. When implementing features with DatabaseKit, follow these guidelines to ensure proper integration and data management.

### Setup and Configuration

1. **Environment Object**:
   - Always access the database through the shared `DB` environment object
   - The `DB` object is attached to the root view and accessible throughout the app
   - Include the environment object in your views:
   ```swift
   struct YourView: View {
       @EnvironmentObject var db: DB
       
       var body: some View {
           // Your view content
       }
   }
   ```

2. **Provider-Specific Implementation**:
   - **Firebase**: Uses Firestore for document-based NoSQL storage
   - **Supabase**: Uses PostgreSQL with Row Level Security policies
   - Configuration settings are handled automatically by SwiftyLaunch

3. **Security Rules**:
   - **Firebase**: Configure security rules in the Firebase Console to control read/write access
   - **Supabase**: Use Row Level Security (RLS) policies to restrict access based on user authentication
   - Create rules that match your app's data access patterns

### Basic Database Operations

1. **Creating Documents**:
   - Use the `addDocument` method to create new documents:
   ```swift
   func createDocument() async {
       let data: [String: Any] = [
           "title": "New Document",
           "content": "Document content",
           "createdAt": Date(),
           "userId": db.auth.currentUser?.uid ?? "anonymous"
       ]
       
       do {
           let documentRef = try await db.collection("yourCollection").addDocument(data: data)
           print("Document created with ID: \(documentRef.documentID)")
       } catch {
           print("Error creating document: \(error)")
       }
   }
   ```

2. **Reading Documents**:
   - Fetch individual documents or collections:
   ```swift
   // Fetch a single document
   func fetchDocument(id: String) async {
       do {
           let document = try await db.collection("yourCollection").document(id).getDocument()
           if document.exists {
               let data = document.data()
               // Process document data
           }
       } catch {
           print("Error fetching document: \(error)")
       }
   }
   
   // Fetch a collection
   func fetchCollection() async {
       do {
           let snapshot = try await db.collection("yourCollection").getDocuments()
           let documents = snapshot.documents.map { document in
               return document.data()
           }
           // Process documents
       } catch {
           print("Error fetching collection: \(error)")
       }
   }
   ```

3. **Updating Documents**:
   - Update existing documents with new data:
   ```swift
   func updateDocument(id: String) async {
       do {
           try await db.collection("yourCollection").document(id).updateData([
               "title": "Updated Title",
               "updatedAt": Date()
           ])
       } catch {
           print("Error updating document: \(error)")
       }
   }
   ```

4. **Deleting Documents**:
   - Remove documents from the database:
   ```swift
   func deleteDocument(id: String) async {
       do {
           try await db.collection("yourCollection").document(id).delete()
       } catch {
           print("Error deleting document: \(error)")
       }
   }
   ```

### Authentication Integration

1. **User-Specific Data**:
   - Store user ID with documents for ownership tracking:
   ```swift
   let userId = db.auth.currentUser?.uid
   ```

2. **Authenticated Operations**:
   - Use the `executeIfSignedIn` helper function to ensure user authentication:
   ```swift
   await db.executeIfSignedIn(withUserID: post.postUserID) {
       await db.deletePost(id: post.id!)
   }
   ```

3. **Security Rules**:
   - Create security rules that validate user ownership:
   ```
   // Firebase example
   match /documents/{documentId} {
     allow read: if true;
     allow write: if request.auth != null && request.auth.uid == resource.data.userId;
   }
   ```

### Data Modeling

1. **Document Structure**:
   - Create Swift structs that match your database structure:
   ```swift
   struct PostData: Codable, Identifiable, Equatable {
       @DocumentID var id: String?
       let title: String
       let content: String
       let creationDate: Date
       let postUserID: String?
   }
   ```

2. **Firestore Compatibility**:
   - Use the `@DocumentID` property wrapper for document IDs
   - Implement `Codable` for automatic serialization/deserialization

3. **State Management**:
   - Use `@Published` properties in your DB class to notify views of data changes:
   ```swift
   @MainActor
   public class DB: ObservableObject {
       @Published var posts: [PostData] = []
   }
   ```

### Query Patterns

1. **Filtering Data**:
   - Filter documents based on specific criteria:
   ```swift
   db.collection("posts")
     .whereField("category", isEqualTo: "technology")
     .order(by: "creationDate", descending: true)
     .limit(to: 10)
   ```

2. **Real-time Updates**:
   - Use listeners to receive real-time updates:
   ```swift
   db.collection("posts").addSnapshotListener { snapshot, error in
       guard let documents = snapshot?.documents else {
           print("Error fetching documents: \(String(describing: error))")
           return
       }
       
       self.posts = documents.compactMap { document in
           try? document.data(as: PostData.self)
       }
   }
   ```

3. **Pagination**:
   - Implement pagination for large collections:
   ```swift
   func loadMorePosts(lastDocument: DocumentSnapshot) async {
       do {
           let nextSnapshot = try await db.collection("posts")
               .order(by: "creationDate", descending: true)
               .start(afterDocument: lastDocument)
               .limit(to: 10)
               .getDocuments()
               
           // Process next batch of documents
       } catch {
           print("Error fetching more documents: \(error)")
       }
   }
   ```

### Integration with Other Modules

1. **With AnalyticsKit**:
   - Database operations are automatically tracked if AnalyticsKit is enabled:
   ```swift
   // This is tracked automatically with source: "db"
   await db.addPost(title: "New Post", content: "Content here")
   ```

2. **With AuthKit**:
   - User authentication state affects database access:
   ```swift
   // Add a post tied to the current user
   func addSignedPost() async {
       guard let userId = db.auth.currentUser?.uid else {
           // Handle unauthenticated state
           return
       }
       
       // Create post with user ID
   }
   ```

3. **With BackendKit**:
   - Access the database from Cloud Functions:
   ```typescript
   // Server-side function
   export const createServerSideDocument = onCall(async (request) => {
     if (!request.auth) {
       throw new Error("Unauthenticated");
     }
     
     // Add document to Firestore
     const document = await firestore.collection("collection").add({
       userId: request.auth.uid,
       // other document data
     });
     
     return { id: document.id };
   });
   ```

### Performance Considerations

1. **Data Structure**:
   - Organize data in logical collections/tables
   - Use subcollections for related data (Firebase)
   - Consider denormalization for frequently accessed data

2. **Query Optimization**:
   - Add indexes for complex queries to improve performance
   - Keep queries as specific as possible to minimize data transfer
   - Use compound queries when appropriate

3. **Offline Capabilities**:
   - Enable offline persistence for better user experience:
   ```swift
   // Enable offline persistence (typically in AppDelegate)
   let settings = FirestoreSettings()
   settings.isPersistenceEnabled = true
   Firestore.firestore().settings = settings
   ```

4. **Batch Operations**:
   - Use batch operations for multiple writes:
   ```swift
   let batch = db.batch()
   
   // Add multiple operations to the batch
   batch.setData(data1, forDocument: ref1)
   batch.updateData(data2, forDocument: ref2)
   batch.deleteDocument(ref3)
   
   // Commit all operations atomically
   try await batch.commit()
   ```

### Implementation Examples

1. **Fetching Posts**:
   ```swift
   @MainActor
   public func fetchPosts() async {
       do {
           let postsSnapshot = try await db.collection("posts").getDocuments()
           self.posts = postsSnapshot.documents.compactMap { document in
               try? document.data(as: PostData.self)
           }
       } catch {
           print("Error fetching posts: \(error)")
       }
   }
   ```

2. **Creating a Post**:
   ```swift
   public func addPost(title: String, content: String) async -> Bool {
       do {
           let newPost = PostData(
               id: nil,
               title: title,
               content: content,
               creationDate: Date(),
               postUserID: "Anonymous"
           )
           
           _ = try await db.collection("posts").addDocument(from: newPost)
           return true
       } catch {
           print("Error adding post: \(error)")
           return false
       }
   }
   ```

3. **Creating a Post with User ID**:
   ```swift
   public func addSignedPost(title: String, content: String) async -> Bool {
       guard let userId = auth.currentUser?.uid else {
           return false
       }
       
       do {
           let newPost = PostData(
               id: nil,
               title: title,
               content: content,
               creationDate: Date(),
               postUserID: userId
           )
           
           _ = try await db.collection("posts").addDocument(from: newPost)
           return true
       } catch {
           print("Error adding signed post: \(error)")
           return false
       }
   }
   ```

4. **Deleting a Post**:
   ```swift
   public static func deletePost(id: String) async -> Bool {
       do {
           try await db.collection("posts").document(id).delete()
           return true
       } catch {
           print("Error deleting post: \(error)")
           return false
       }
   }
   ```

### Testing

1. **Local Testing**:
   - Use Firebase/Supabase Emulator for local testing
   - Create mock data for unit tests
   - Test database security rules thoroughly

2. **Mock Responses**:
   - Implement mock DB class for testing:
   ```swift
   class MockDB: DBProtocol {
       var posts: [PostData] = []
       
       func fetchPosts() async {
           // Return mock data
           self.posts = [
               PostData(id: "1", title: "Test Post", content: "Test Content", creationDate: Date(), postUserID: "test-user")
           ]
       }
       
       // Implement other methods
   }
   ```

3. **Error Handling**:
   - Test various error scenarios to ensure robust error handling
   - Validate transaction atomicity in batch operations

### Running Locally

1. **Firebase Emulator Setup**:
   - Install Firebase CLI tools
   - Configure emulator for local development:
   ```bash
   firebase init emulators
   ```

2. **Admin Credentials**:
   - Set up service account key for local development
   - Configure environment variables:
   ```bash
   export GOOGLE_APPLICATION_CREDENTIALS="path/to/key.json"
   ```

3. **Local Database Configuration**:
   - Configure your app to use local emulator:
   ```swift
   // In FirebaseBackend.swift
   let settings = Firestore.firestore().settings
   settings.host = "localhost:8080"
   settings.isPersistenceEnabled = false
   settings.isSSLEnabled = false
   Firestore.firestore().settings = settings
   ```

### Troubleshooting

1. **Authentication Errors**:
   - For "UNAUTHENTICATED" errors with Firebase Functions:
     - Go to Cloud Run Functions in Google Cloud Console
     - Add `allUsers` principal with "Cloud Run Invoker" role
     - Allow public access if needed

2. **Data Access Issues**:
   - Verify security rules in Firebase Console
   - Check user authentication state
   - Review database structure and query patterns

3. **Performance Problems**:
   - Add appropriate indexes for complex queries
   - Monitor database usage quotas
   - Implement pagination for large datasets
