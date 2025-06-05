# InstaRetro :: Ultimate Edition

InstaRetro is a feature-rich, single-page web application designed to mimic the core functionalities of Instagram. It's built using React (via CDN) for the frontend and Firebase for backend services, including authentication, database (Firestore), and file storage. This "Ultimate Edition" aims to provide a comprehensive set of features found in modern social media applications.

## Features

*   **Authentication:**
    *   User Sign-up (Email/Password, Username, Full Name)
    *   User Log-in
    *   User Log-out
*   **Feed & Posts:**
    *   Infinite scroll-style home feed displaying posts from followed users (currently shows all posts, feed algorithm not implemented).
    *   Create new posts (image or video) with captions and optional location.
    *   View individual posts.
    *   Like/Unlike posts.
    *   Save/Unsave posts (bookmarking).
    *   Double-tap to like media.
*   **Comments:**
    *   View comments on posts.
    *   Add new comments (with @mention detection for potential notifications).
    *   Real-time comment updates.
*   **Stories:**
    *   View stories from followed users (and own) in a top bar.
    *   Create new image/video stories (expire after 24 hours, basic implementation).
    *   View stories in a full-screen modal.
*   **User Profiles:**
    *   View user profiles with avatar, username, full name, bio.
    *   Display user's posts in a grid.
    *   Show follower/following counts and post count.
    *   Edit own profile (avatar, full name, bio).
    *   Follow/Unfollow other users.
*   **Search:**
    *   Search for users by username (basic prefix search).
*   **Activity/Notifications:**
    *   Real-time activity feed for likes, new followers, comments, and mentions.
    *   Visual indication for unread notifications.
    *   Clicking notifications navigates to the relevant content (post or profile).
*   **Direct Messaging (Chat):**
    *   View a list of ongoing conversations.
    *   Initiate new chats from user profiles.
    *   Real-time one-on-one messaging.
    *   Sent/Received message bubbles.
*   **Modals:**
    *   Used for creating posts/stories, viewing comments, editing profiles, viewing story details, and post details.
*   **UI/UX:**
    *   Responsive design emulating a mobile app interface.
    *   Customizable theme via CSS variables.
    *   Loading indicators for asynchronous operations.

## Technologies Used

*   **Frontend:**
    *   HTML5
    *   CSS3 (with CSS Variables for theming)
    *   JavaScript (ES6+)
    *   React 17 (via CDN: `react.development.js`, `react-dom.development.js`)
    *   Babel Standalone (via CDN: `@babel/standalone` for JSX transpilation in the browser)
    *   UUID (via CDN for generating unique IDs)
*   **Backend (Firebase - v9 Compat SDKs):**
    *   Firebase Authentication: For user sign-up, sign-in, and session management.
    *   Firebase Firestore: NoSQL database for storing user data, posts, comments, likes, follows, stories, notifications, and chat messages.
    *   Firebase Storage: For storing user-uploaded images and videos (profile pictures, post media, story media).
*   **Fonts:**
    *   Billabong (Instagram-like logo font)
    *   San Francisco (System font)

## Prerequisites

*   A modern web browser (Chrome, Firefox, Safari, Edge).
*   An active internet connection.
*   A Firebase project.

## Setup and Installation

1.  **Clone or Download the Repository:**
    If this were a Git repository:
    ```bash
    git clone <repository-url>
    cd instaretro-ultimate-edition
    ```
    Otherwise, simply download the `index.html` file.

2.  **Firebase Project Setup:**
    *   Go to the [Firebase Console](https://console.firebase.google.com/).
    *   Click on "Add project" and follow the steps to create a new project.
    *   Once your project is created, navigate to Project Settings (click the gear icon next to "Project Overview").
    *   Under the "General" tab, scroll down to "Your apps".
    *   Click on the Web icon (`</>`) to add a web app.
    *   Register your app (give it a nickname, e.g., "InstaRetro Web"). You **do not** need to set up Firebase Hosting at this stage if you're just running it locally.
    *   After registering, Firebase will provide you with a `firebaseConfig` object. **Copy this object.**

3.  **Configure Firebase in `index.html`:**
    *   Open the `index.html` file in a text editor.
    *   Find the `firebaseConfig` constant (around line 200):
        ```javascript
        // IMPORTANT: Replace with your Firebase project configuration
        /*const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_AUTH_DOMAIN",
            projectId: "YOUR_PROJECT_ID",
            storageBucket: "YOUR_STORAGE_BUCKET",
            messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
            appId: "YOUR_APP_ID"
        };*/
        const firebaseConfig = { /* ... current placeholder config ... */ };
        ```
    *   **Replace the placeholder `firebaseConfig` object with the one you copied from your Firebase project.**

4.  **Enable Firebase Services:**
    In the Firebase Console for your project:
    *   **Authentication:**
        *   Go to "Authentication" (Build section).
        *   Click the "Sign-in method" tab.
        *   Enable "Email/Password" provider.
    *   **Firestore Database:**
        *   Go to "Firestore Database" (Build section).
        *   Click "Create database".
        *   Start in **Test mode** for initial development. (Click "Next", then choose a region, then "Enable").
        *   **Important:** For production, you MUST set up proper [Security Rules](#firebase-security-rules).
    *   **Storage:**
        *   Go to "Storage" (Build section).
        *   Click "Get started".
        *   Choose "Start in test mode" for initial development. (Click "Next", then "Done").
        *   **Important:** For production, you MUST set up proper [Security Rules](#firebase-security-rules).

5.  **Firebase Security Rules (Crucial for Production):**
    The default "test mode" rules allow open access. You **must** update these for any real deployment. The code comments provide example rules. You'll need to paste these into the "Rules" tab of Firestore and Storage respectively in the Firebase console.

    **Example Firestore Rules (from code comments, adapt as needed):**
    ```firestore-rules
    rules_version = '2';
    service cloud.firestore {
      match /databases/{database}/documents {
        match /users/{userId} {
          allow read;
          allow write: if request.auth.uid == userId;
          // Allow creating a user doc during signup even if not authenticated yet
          allow create: if request.auth == null && request.resource.data.uid == request.auth.uid; // This needs adjustment for signup
          // A better rule for user creation during signup might be:
          // allow create: if request.resource.data.uid == request.auth.uid; (after auth happens)
          // Or if user doc is created by a cloud function post-signup
        }
        match /posts/{postId} {
          allow read;
          allow create: if request.auth.uid == request.resource.data.userId;
          allow update, delete: if request.auth.uid == resource.data.userId;
        }
        match /likes/{likeId} { // likeId is typically userId_postId
          allow read;
          allow create, delete: if request.auth != null && request.auth.uid == request.resource.data.userId;
        }
        match /saved_posts/{saveId} { // saveId is typically userId_postId
          allow read;
          allow create, delete: if request.auth != null && request.auth.uid == request.resource.data.userId;
        }
        match /comments/{commentId} {
          allow read;
          allow create: if request.auth != null && request.auth.uid == request.resource.data.userId;
          // Add rules for update/delete if needed
        }
        match /followers/{followId} { // followId is typically followerId_followingId
          allow read;
          allow create, delete: if request.auth != null && request.auth.uid == request.resource.data.followerId;
        }
        match /stories/{storyId} {
          allow read;
          allow create: if request.auth != null && request.auth.uid == request.resource.data.userId;
          // Add delete rule if users can delete stories, or use TTL policies
        }
        match /notifications/{notificationId} {
          allow read: if request.auth != null && request.auth.uid == resource.data.userId;
          allow create: if request.auth != null; // Action triggered by an authenticated user
          allow update(isRead): if request.auth != null && request.auth.uid == resource.data.userId;
        }
        match /chatRooms/{chatRoomId} {
          allow read, write: if request.auth != null && request.auth.uid in resource.data.participants;
          allow create: if request.auth != null && request.auth.uid in request.resource.data.participants;
        }
        match /chatRooms/{chatRoomId}/messages/{messageId} {
          allow read, create: if request.auth != null && get(/databases/$(database)/documents/chatRooms/$(chatRoomId)).data.participants.hasAny([request.auth.uid]);
        }
      }
    }
    ```

    **Example Storage Rules (from code comments, adapt as needed):**
    ```storage-rules
    rules_version = '2';
    service firebase.storage {
      match /b/{bucket}/o {
        match /{allPaths=**} {
          allow read; // Allow public read for images/videos typically
        }
        // Restrict writes to authenticated users and specific paths/file types/sizes
        match /profile_pics/{userId}/{fileName} {
          allow write: if request.auth != null && request.auth.uid == userId
                       && request.resource.size < 5 * 1024 * 1024 // Max 5MB
                       && request.resource.contentType.matches('image/.*');
        }
        match /posts/{userId}/{fileName} {
          allow write: if request.auth != null && request.auth.uid == userId
                       && request.resource.size < 20 * 1024 * 1024 // Max 20MB for posts (consider video)
                       && request.resource.contentType.matches('image/.*|video/.*');
        }
        match /stories/{userId}/{fileName} {
          allow write: if request.auth != null && request.auth.uid == userId
                       && request.resource.size < 20 * 1024 * 1024 // Max 20MB for stories
                       && request.resource.contentType.matches('image/.*|video/.*');
        }
      }
    }
    ```

6.  **Firestore Indexes:**
    Certain queries, especially those involving `orderBy` on one field and `where` on another, or `in` queries with `orderBy`, require composite indexes. Firebase usually prompts you to create these in the console when a query fails due to missing indexes.
    Based on the code, you might need indexes like:
    *   `posts`: `userId` (ASC), `createdAt` (DESC)
    *   `stories`: `userId` (ASC), `createdAt` (ASC/DESC depending on query)
    *   `stories`: `userId` (`array-contains` or `in`), `createdAt` (DESC)
    *   `notifications`: `userId` (ASC), `createdAt` (DESC)
    *   `chatRooms`: `participants` (`array-contains`), `lastMessageTimestamp` (DESC)
    *   `users`: `username` (ASC) (for username search with range queries)

## Running the Application

*   Simply open the `index.html` file in your web browser.
    *   On most systems, you can double-click the file.
    *   Or, right-click and choose "Open with" your preferred browser.
    *   Alternatively, you can drag and drop the `index.html` file onto an open browser window.

## Key Data Structures (Firestore Collections)

*   **`users`**: Stores user profile information.
    *   `uid`, `username`, `email`, `fullname`, `profilePicUrl`, `bio`, `createdAt`, `followersCount`, `followingCount`, `postsCount`.
*   **`posts`**: Stores information about each post.
    *   `userId`, `contentUrl`, `contentType` (image/video), `thumbnailUrl`, `caption`, `location`, `likesCount`, `commentsCount`, `createdAt`.
*   **`likes`**: Tracks likes on posts (document ID: `userId_postId`).
    *   `userId`, `postId`, `createdAt`.
*   **`saved_posts`**: Tracks posts saved by users (document ID: `userId_postId`).
    *   `userId`, `postId`, `savedAt`.
*   **`comments`**: Stores comments for posts.
    *   `postId`, `userId`, `text`, `createdAt`.
*   **`followers`**: Manages follow relationships (document ID: `followerId_followingId`).
    *   `followerId`, `followingId`, `createdAt`.
*   **`stories`**: Stores user stories.
    *   `userId`, `contentUrl`, `contentType`, `createdAt`.
*   **`notifications`**: Stores notifications for users.
    *   `userId` (recipient), `actorId` (who performed action), `type` (like, comment, follow, mention), `postId`, `commentId`, `message`, `isRead`, `createdAt`.
*   **`chatRooms`**: Represents a conversation between two users (document ID: `sortedUserId1_sortedUserId2`).
    *   `participants` (array of two user UIDs), `createdAt`, `lastMessageText`, `lastMessageSenderId`, `lastMessageTimestamp`.
*   **`chatRooms/{chatRoomId}/messages`**: Subcollection storing individual messages within a chat room.
    *   `senderId`, `text`, `createdAt`.

## Future Enhancements / Known Issues

*   **Feed Algorithm:** Implement a proper feed algorithm (e.g., show posts from followed users only, sorted chronologically or by relevance).
*   **Advanced Search:** Firestore's native search is limited. For more robust search (e.g., full-text, typo tolerance), integrate a third-party service like Algolia or Elasticsearch.
*   **Video Thumbnails:** Implement server-side (e.g., via Firebase Cloud Functions) or more robust client-side video thumbnail generation.
*   **Pagination:** Implement pagination for feeds, comment lists, search results, etc., to handle large datasets efficiently and reduce Firestore read costs.
*   **Story Features:**
    *   Auto-deletion of stories after 24 hours (e.g., using Firestore TTL policies or Cloud Functions).
    *   Story viewers list.
    *   Interactive elements in stories.
*   **Real-time Updates:** While some parts are real-time (comments, chat, notifications), others like follower counts on profiles or like counts on posts in the feed might only update on refresh/re-fetch.
*   **Error Handling:** Improve error handling with user-friendly messages instead of generic alerts.
*   **UI/UX Refinements:** Further polish the UI/UX, add animations, and improve accessibility.
*   **Testing:** Implement unit and integration tests.
*   **Cloud Functions:**
    *   For complex backend logic (e.g., sending aggregated notifications, cleaning up data, generating thumbnails).
    *   More robust @mention notification system.
*   **Saved Posts Tab:** Fully implement the "Saved Posts" tab on the profile page.
*   **Code Structure:** For larger applications, consider splitting React components into separate `.jsx` files and using a build system (like Create React App, Vite, or Next.js) rather than CDNs and inline Babel.
