# From Complaints to Action

A responsive civic complaint demo created for **PromptWars × ASTRA — AI for Civic Impact**. Citizens can submit a report, see transparent rule-based analysis, follow complaint progress, and explore community patterns. The interface uses Firebase when configured and otherwise labels browser-local demo storage clearly.

## Problem statement

Civic concerns can be difficult to report and follow. Residents need a simple way to describe a problem and check its progress, while civic teams need a consistent view of issue types, urgency, location, assignment, and resolution. This project demonstrates a connected workflow for those needs.

## Objectives

- Capture complaints with a title, description, category, address, optional coordinates, and optional photo.
- Summarize and classify reports, estimate priority with an explanation, and flag possible duplicates.
- Reveal category and priority patterns and nearby report clusters without presenting synthetic data as real.
- Demonstrate department assignment, status updates, activity history, and citizen tracking.
- Provide a rule-based Civic Assistant that answers common questions and looks up actual saved complaint IDs.

## Technologies

- React and JavaScript for the interface and application logic.
- Vite for the development server and production build.
- CSS for responsive layouts and component styling.
- Leaflet with OpenStreetMap tiles for interactive complaint location selection and Reveal markers.
- Firebase Firestore and Firebase Storage as optional hosted persistence.
- Node.js built-in test runner for automated service and rule tests.

## Architecture

```text
src/
  ai/          Rule-based analyzer and clearly labelled synthetic records
  components/  Shared navigation, charts, map, analysis card, chat UI
  firebase/    Firebase Web SDK configuration and initialization
  pages/       Overview, Capture, Reveal, Admin Management, Track Complaint
  services/    Persistence, analysis workflow, analytics, assistant replies
  styles/      Feature-specific CSS
  App.jsx      Hash navigation and shared assistant
  main.jsx     React entry point
tests/         Node tests for storage, analysis, analytics, workflow, assistant
```

`complaintService` is the boundary between pages and persistence. With Firebase configuration it uses the `complaints` Firestore collection and Firebase Storage for photos. Without it, complaint records (including demo image data URLs) are kept in this browser's local storage. `complaintAnalyzer`, `complaintAnalytics`, `complaintWorkflow`, and `civicAssistant` contain reusable logic that can be tested without rendering the UI.

## Modules

- **Capture:** Register a complaint with required title, description, category, and address. Preview an optional photo, add optional latitude and longitude, and receive a unique complaint ID after a successful save.
- **Understand:** Shows category, short summary, possible duplicate matches, priority, and the reason for that priority.
- **Reveal:** Shows counts, category and priority charts, recent reports, an interactive OpenStreetMap with selectable citizen report markers, and repeated-location groups. Selecting a marker shows the report's ID, title, category, priority, status, address, and coordinates; private respondent details are not collected. The 40+ generated records are labelled synthetic and never plotted as real locations.
- **Act / Admin Management:** Search and filter reports, assign a department and staff member, update status, add notes or resolution details, and review the activity history. Synthetic examples are read-only. The current page has no administrator authentication, so it is a workflow demo and must not be exposed as a production admin tool.
- **Track Complaint:** Looks up an entered ID and shows saved citizen-facing progress, department, timeline, notes, and resolution details. Unknown IDs receive a not-found message.
- **Civic Assistant:** The floating panel answers the suggested civic questions, handles actual saved complaint ID lookups, and avoids exposing assigned staff names or admin notes. It is a rule-based demo assistant, not a connected AI model.

## GPS location in Capture

On the complaint registration page, use the **GPS Location Tracker** and choose **Detect My Current Location** to trigger the browser's geolocation permission prompt and request a one-time position. Location is not requested on page load and is not collected in the background. On success, latitude and longitude fill the editable coordinate fields and the OpenStreetMap view centers a marker there. You can pan and zoom, then click the map to place or adjust the marker; manual coordinate edits also move it. Choose **Clear Location** to remove coordinates and the marker. Both coordinates must be supplied together and stay within latitude `-90..90` and longitude `-180..180`. The form does not submit automatically; review and submit the complaint yourself. Browser geolocation requires a secure context, such as `localhost` during development or HTTPS when deployed, and permission from the user. The interactive map loads OpenStreetMap tiles while Capture is open and while Reveal has coordinate reports to display. Those are normal map requests to OpenStreetMap; geolocation coordinates are not requested unless the citizen clicks the location button. Submitted numeric coordinates are saved in the existing Firestore/local demo record and shown on Reveal only for non-synthetic complaints with valid coordinates.

## Analysis and sample-data honesty

Complaint analysis is deterministic keyword and heuristic logic, not a real AI model. It infers a category from the report text, creates a short extractive summary, compares category, description, and location for possible duplicates, and assigns one of Critical, High, Medium, or Low with a human-readable reason. Results are suggestions and can be imperfect. No external AI API is configured.

Dashboard sample reports are synthetic test/demo data. They are not evidence of real civic reports, real residents, real incidents, or verified real-world locations. They are explicitly labelled in the interface and excluded from coordinate map pins.

## Requirements and installation

Install Node.js (npm is included), then open a terminal in the project folder. Install dependencies:

```sh
npm install
```

Start the local development server:

```sh
npm run dev
```

Open the local URL printed by Vite (normally `http://localhost:5173`). Stop the server with `Ctrl+C`.

Create and inspect the production build:

```sh
npm run build
npm run preview
```

Run automated tests:

```sh
npm test
```

The tests cover local demo storage and unique IDs, category and priority analysis, duplicate detection, dashboard counts and hotspots, assignment/status/history/tracking helpers, assistant suggested replies and ID lookup/privacy behavior, GPS coordinate validation and browser geolocation success/error handling, coordinate preservation in local and Firestore payload builders, map marker coordinate filtering, and narrow-screen chat CSS rules. These are automated logic tests; they do not replace manual browser testing of the permission prompt, interactive map tiles and clicks, photo selection, visual layout, or an actual Firebase project.

Optional static analysis:

```sh
npm run lint
```

## Firebase setup

Firebase is optional for local demo use. The checked-in `.env.example` contains placeholder names and values only; it is not a credential file and does not connect to a Firebase project.

1. Create a Firebase project and register a Web app in the Firebase console.
2. Copy `.env.example` to `.env.local` and replace the placeholders with that Web app's configuration.
3. Enable Firestore and (if uploading images) Firebase Storage. Set rules appropriate for your intended use.
4. Restart the Vite server after changing environment values.

The app requires `VITE_FIREBASE_API_KEY`, `VITE_FIREBASE_PROJECT_ID`, and `VITE_FIREBASE_APP_ID` to select Firebase mode. Storage uploads additionally require `VITE_FIREBASE_STORAGE_BUCKET`. A Firebase web configuration is public client configuration, not a server secret; do not put service-account keys, private API secrets, or AI provider secrets in `VITE_` variables. The UI only reports configured mode; an actual successful read or write also depends on network access, enabled services, and correct Firebase rules. Failed complaint reads show an error instead of silently switching to local records. Local browser records are separate from Firestore and do not sync between devices.

## Limitations and future improvements

- Admin Management currently has no sign-in or role-based authorization. Protect admin routes and enforce Firestore security rules before any real deployment; hiding a link is not authorization.
- Firebase rules, project provisioning, and live Firebase integration are not included or verified without the owner's Firebase project and credentials.
- Local demo storage is browser- and device-specific and may be cleared by the user or browser. Image data URLs consume local storage space.
- The rule-based category, summary, duplicate, and priority logic can misclassify reports and should be reviewed by civic staff.
- Synthetic reports are for demonstrations only. Map pins require actual submitted coordinates; there is no geocoder or map tile service.
- The assistant is deterministic and limited to the supported civic topics. It is not a general AI assistant and should not be used for emergency response.
- Future work could add authentication and roles, verified department workflows, accessible geocoding/maps, server-side AI behind a protected API, data retention/privacy controls, and end-to-end browser tests.
