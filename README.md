# Chasni: Local-First Staff Automation Module (`chasni-task-runner`)

A lightweight, local-first Android utility built for small sweet shop (*mithai*) owners. This module automates internal shop operations and staff communication using zero-cost on-device technologies. It translates generic UI actions from the store owner into localized, dynamic voice commands directed at specific active employees (e.g., announcing out loud: *"Subbu, check the inventory"*), without forcing the owner to manually input names during busy shop hours.

---

## 🚀 Core Features
*   **Dynamic Role-to-Name Mapping:** Automatically queries the local database to find which employee is currently assigned to a role (e.g., mapping `kitchen_helper` to `"Subbu"`).
*   **On-Device Text-to-Speech (TTS):** Uses native Android TTS engines with an `en-IN` (Indian English) or regional language locale. It provides clear, hands-free speaker announcements inside noisy kitchens or counters with zero external API costs.
*   **Zero-Budget Architecture:** Operates 100% offline using a local SQLite/Room state layer. It runs on entry-level Android devices without requiring paid servers or cloud configurations.

---

## 🛠️ Tech Stack & Constraints
*   **Language/Framework:** Kotlin (Native Android) / Flutter (Dart)
*   **Local Database:** Room Persistence Library / SQLite (Offline-first data layer)
*   **Audio Pipeline:** Native `android.speech.tts.TextToSpeech` API
*   **Hardware Target:** Entry-level Android devices (RAM $\ge$ 2GB, Android 8.0+)

---

## 🗄️ Local Database Schema

The module relies on a single-table state tracking mechanism to resolve personnel names dynamically:

