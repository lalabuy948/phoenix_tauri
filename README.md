# PhoenixTauri

A desktop application powered by Phoenix LiveView and Tauri. Packages a Phoenix backend as a standalone executable via Burrito, wrapped in a native Tauri window.

![](/preview.png)

![](/dmg.png)

## Prerequisites

- Elixir 1.15+ / Erlang OTP 27+
- Rust 1.70+ and Cargo
- Node.js 18+ / npm
- Zig 0.15.2+
- Tauri CLI (`cargo install tauri-cli`)

## Development

Start the Phoenix server for local development:

```bash
mix setup
mix phx.server
```

Visit [localhost:4000](http://localhost:4000) in your browser.

## Building the Desktop App (.dmg / .app)

### Step 1: Install dependencies

```bash
# Elixir deps
mix deps.get

# Tauri npm deps
cd tauri && npm install && cd ..
```

### Step 2: Build Phoenix assets for production

```bash
MIX_ENV=prod mix assets.deploy
```

### Step 3: Build the Burrito release (standalone Phoenix binary)

```bash
MIX_ENV=prod mix release phoenix_tauri_desktop
```

This generates platform-specific binaries in `burrito_out/`:
- `phoenix_tauri_desktop_macos` (macOS ARM64)
- `phoenix_tauri_desktop_linux` (Linux x86_64)
- `phoenix_tauri_desktop_windows.exe` (Windows x86_64)

### Step 4: Link the sidecar binary for Tauri

Create a symlink so Tauri can find the Phoenix backend binary. On macOS (Apple Silicon):

```bash
cd tauri/src-tauri
ln -sf ../../burrito_out/phoenix_tauri_desktop_macos \
   phoenix_tauri_backend-aarch64-apple-darwin
cd ../..
```

For other platforms:
```bash
# Linux x86_64
ln -sf ../../burrito_out/phoenix_tauri_desktop_linux \
   phoenix_tauri_backend-x86_64-unknown-linux-gnu

# Windows x86_64
ln -sf ../../burrito_out/phoenix_tauri_desktop_windows.exe \
   phoenix_tauri_backend-x86_64-pc-windows-msvc.exe
```

### Step 5: Build the Tauri application

```bash
cd tauri
npx tauri build
```

The built application will be at:
- **macOS .app**: `tauri/src-tauri/target/release/bundle/macos/Phoenix Tauri.app`
- **macOS .dmg**: `tauri/src-tauri/target/release/bundle/dmg/Phoenix Tauri_0.1.0_aarch64.dmg`

### Quick build (all steps combined)

```bash
MIX_ENV=prod mix assets.deploy && \
MIX_ENV=prod mix release phoenix_tauri_desktop && \
cd tauri/src-tauri && \
ln -sf ../../burrito_out/phoenix_tauri_desktop_macos phoenix_tauri_backend-aarch64-apple-darwin && \
cd .. && \
npx tauri build && \
cd .. && \
mix phx.digest.clean --all
```

If you don't do proper versioning of backend you might have cached sidecard, to delete it on mac you can:

```zsh
rm -rf ~/Library/Application\ Support/.burrito/  
```

## Architecture

```
Tauri (Rust native window)
  └── launches Phoenix backend as sidecar process
        └── Phoenix serves on localhost:4000
              └── Tauri webview connects to localhost:4000
                    └── Phoenix LiveView handles all UI
```

## Project Structure

```
phoenix_tauri/
├── lib/                     # Elixir/Phoenix application
│   ├── phoenix_tauri/       # Business logic, Repo, Release
│   └── phoenix_tauri_web/   # Web layer, LiveViews, router
├── assets/                  # Frontend (JS, CSS, vendor libs)
├── config/                  # Elixir configuration
├── tauri/                   # Tauri desktop wrapper
│   ├── src/                 # Loading page HTML
│   ├── src-tauri/           # Rust source, Cargo.toml, config
│   └── package.json         # Tauri npm dependencies
└── mix.exs                  # Elixir project definition
```

## Learn more

- Phoenix: https://hexdocs.pm/phoenix/overview.html
- Tauri: https://tauri.app/
- Burrito: https://github.com/burrito-elixir/burrito
- Blog post: https://mrpopov.com/posts/elixir-liveview-single-binary/
