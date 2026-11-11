# otp_demo — test app for serverpod/serverpod#5874

Throwaway fullstack project used to test the verification code input fix
(Marcelo's `pin-codes` branch) on iPhone Safari and desktop web.

## What is in here

- Standard `serverpod create otp_demo -t fullstack --auth` output, with the
  Flutter app's `main.dart` wrapping `GreetingsScreen` in the template's
  `SignInScreen` so the email sign-in flow shows on launch.
- `vendor/pinput`: a copy of pinput 6.0.2 with one change in
  `lib/src/utils/pinput_constants.dart`, the hidden `EditableText` font size
  raised from 1px to 16px. iOS Safari zooms on tap for any input under 16px,
  which is the suspected cause of the zoom-on-tap in #5874. Remove the
  `pinput` entry under `dependency_overrides` in the root `pubspec.yaml` to
  test against the unpatched package.
- `otp_demo_server/config/passwords.yaml` is committed on purpose. These are
  the generated local-dev secrets for the docker Postgres and Redis in this
  repo, nothing else.

## Point the overrides at your serverpod checkout

The root `pubspec.yaml` has `dependency_overrides` with absolute paths into a
serverpod monorepo checkout on the `pin-codes` branch
(`github.com/marcelomendoncasoares/serverpod`). Repoint them to yours:

```
sed -i '' 's#/Users/preston/projects/orca/workspaces/serverpod/its-virtually-impossible-to-create-an-account-on-2#/path/to/your/serverpod#g' pubspec.yaml
flutter pub get
```

Run `dart pub get` in the serverpod checkout first so its workspace is
bootstrapped.

## Run

```
cd otp_demo_server
docker compose up -d postgres redis
dart run bin/main.dart --apply-migrations
```

Login codes print to the server console as `Login code for <email>: <123456>`.

Phone test, with the phone on the same Wi-Fi as the Mac:

```
cd otp_demo_flutter
flutter run -d web-server --release --web-port 5874 --web-hostname 0.0.0.0 \
  --dart-define=SERVER_URL=http://<mac-lan-ip>:8080/
```

Then open `http://<mac-lan-ip>:5874` in Safari on the phone. Use `--release`:
a debug build made Safari reload the tab under memory pressure right after
the zoom. Plus-aliased emails are rejected by the sign-in form.

## What to check

- Single tap on the pin field: zooms with unpatched pinput, should not with
  `vendor/pinput`.
- Caret and focused border visible in light and dark mode.
- Clipboard icon next to the pins brings up Safari's Paste bubble and fills
  the code.
- Long-press on the pins shows a menu with Paste.
- Number pad comes up.
- Desktop Chrome: right-click shows Flutter's Paste menu, Cmd+V works.
