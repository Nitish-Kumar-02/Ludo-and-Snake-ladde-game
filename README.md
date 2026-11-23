# Board Arena — Ludo & Snakes and Ladders

A Flutter project with working game logic, boards, online multiplayer
(Firebase), and text chat with emoji for both **Ludo** and
**Snakes and Ladders**. This was written and reviewed as source code in
a sandbox without the Flutter SDK installed, so **treat it as a strong
first draft, not a compiled/tested build** — budget time to run
`flutter pub get`, fix any version-mismatch errors, and polish the
board visuals (see "Known rough edges" below) before you ship it.

## What's actually in here

- `lib/models/ludo_model.dart` — full Ludo rules: 4 tokens/player, entry
  on a 6, capturing, safe cells, home column, extra turn on 6 or a
  capture, three-sixes forfeit.
- `lib/models/snake_ladder_model.dart` — full Snakes & Ladders rules:
  100 squares, snake/ladder table, exact-roll-to-finish, extra turn on 6.
- `lib/widgets/ludo_board.dart`, `lib/widgets/snake_ladder_board.dart` —
  hand-drawn boards (`CustomPainter`), no image assets required.
- `lib/services/room_service.dart` — Firebase Firestore room
  creation/joining with a shareable 6-character code, host-authoritative
  state sync, and chat.
- `lib/widgets/chat_panel.dart` — text chat with an emoji picker,
  bottom-sheet launcher.
- `lib/screens/` — home screen, lobby (host/join), and both game screens
  wired end to end.

**Voice chat is intentionally not included.** It needs a paid real-time
SDK (Agora, Twilio, or ZEGOCLOUD all have free tiers to start) and its
own setup with API keys — see "Adding voice chat later" below for the
path once the rest is working and you have users to justify the cost.

## 1. Set up the Flutter project

1. Install the Flutter SDK (this sandbox doesn't have it, so this repo
   has never been run through `flutter pub get` or `flutter run`):
   https://docs.flutter.dev/get-started/install
2. From this folder: `flutter pub get`
3. `flutter run` (with an emulator or device connected) — expect to fix
   a handful of small issues, since this hasn't been compiled yet.

## 2. Wire up Firebase (required for online multiplayer + chat)

1. Create a project at https://console.firebase.google.com
2. In the project, enable:
   - **Authentication** → Sign-in method → **Anonymous**
   - **Firestore Database** → Create database (start in test mode)
3. Install the FlutterFire CLI and run it from this project folder:
   ```
   dart pub global activate flutterfire_cli
   flutterfire configure
   ```
   This generates `lib/firebase_options.dart` with your project's keys.
   **Do not commit this file to a public GitHub repo** if you want to
   keep your Firebase config private (add it to `.gitignore`).
4. In `lib/main.dart`, uncomment the two lines that reference
   `firebase_options.dart` and `DefaultFirebaseOptions.currentPlatform`.
5. Once you have real users, replace `firestore.rules.sample` content
   into the Firestore "Rules" tab in the console (test-mode rules
   expire after 30 days and are wide open until then).

## 3. Known rough edges to fix before shipping

- **Ludo board geometry** (`_BoardGeometry._verifiedLoop` in
  `lib/widgets/ludo_board.dart`): the 52-cell track and yard positions
  are hand-coordinated to match the classic cross board, but weren't
  rendered/checked visually (no Flutter runtime available here). Run
  the app, roll some dice, and watch whether tokens follow a sensible
  clockwise path — nudge the `Offset` coordinates in that file until
  they do. This is the single most likely thing to need adjustment.
- **Turn-based networking is simple, not bulletproof**: the host device
  runs the real game and everyone else sends "please roll" / "please
  move token X" requests. If the host's app closes mid-game, the room
  stalls — there's no host-migration logic. Fine for friends playing
  together; add reconnect/host-handoff logic before treating this as
  production-grade for strangers.
- **No input validation on the Firestore side beyond the sample rules**
  — e.g. nothing stops a modified client from sending a "move" action
  out of turn. The host code doesn't check whose turn it is before
  applying an action. Add that check in `game_screen_ludo.dart` /
  `game_screen_snake_ladder.dart`'s `_handleHostActions` before launch.
- **No automated tests.** Given the hand-off rules complexity (capturing,
  safe cells, exact-finish), write a few `flutter test` unit tests
  against `LudoGame`/`SnakeLadderGame` directly — they're plain Dart
  classes with no Flutter dependency, so this is quick to do.

## 4. Monetization options (for Play Store revenue)

- **Ads**: `google_mobile_ads` package — banner between games, or a
  rewarded ad for an extra dice roll/undo. Requires an AdMob account.
- **In-app purchases**: `in_app_purchase` package — cosmetic token
  skins, board themes, or "remove ads". Requires Play Console product
  setup.
- Realistically, a board-game app makes money through volume (ads) or
  a loyal niche (cosmetics), not both well at once — pick one to start.

## 5. Publishing to Google Play

1. Create a [Google Play Console](https://play.google.com/console)
   developer account (one-time $25 fee).
2. `flutter build appbundle --release` — you'll need to set up your own
   signing key first (`android/key.properties` +
   `android/app/build.gradle` signing config — Flutter's official docs
   walk through this: https://docs.flutter.dev/deployment/android).
3. Write a privacy policy (required if you collect any data — anonymous
   auth + chat messages count) and host it anywhere public; link it in
   the Play Console listing.
4. Fill out the Data Safety form in Play Console honestly — this app
   collects chat text and a device-generated anonymous ID.
5. Submit for review. First-time accounts sometimes go through an
   extended review period for new developers.

## 6. Adding voice chat later

Once the game works and you have a reason to spend on it:
1. Pick a provider — Agora and ZEGOCLOUD both have generous free tiers
   and Flutter SDKs (`agora_rtc_engine` / `zego_express_engine`).
2. Add a "Join voice" button in the game screen that joins an RTC
   channel named after the room code — the room code you already have
   doubles perfectly as the channel name.
3. Budget for cost: these are billed per participant-minute past the
   free tier, so test your expected concurrent usage against their
   pricing pages before launch.
