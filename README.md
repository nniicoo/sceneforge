# SceneForge

**Browser-playable AI-driven 3D scene generator.**  
Describe a room in natural language → AI generates JSON → Unity instantiates the scene in real time. Walk up to a VRoid avatar NPC for multi-turn conversation-driven scene editing.

🎮 **[Play on itch.io](https://your-username.itch.io/sceneforge)** · 📦 **[Releases](../../releases)**

---

## Demo

> 📸 _Screenshot 1 — Opening view: empty room with UI panel on the left_  
> 📸 _Screenshot 2 — After typing "a cozy library with reading corner": books, desk, sofa appear_  
> 📸 _Screenshot 3 — NPC dialogue panel open, user typing "move the desk to the left wall"_  
> 📸 _Screenshot 4 — Preset selector showing Library / Office / Living Room templates_  
> 🎞️ _GIF — Full loop: type prompt → scene generates → walk to NPC → chat to move objects_

---

## Features

| Feature | Description |
|---|---|
| 🧠 Natural language generation | Type any room description, AI returns structured JSON, scene renders instantly |
| 🗂️ Preset templates | One-click Library / Office / Living Room scenes |
| 💬 NPC conversation editor | Walk near the VRoid avatar, chat to move, rotate, add, or remove objects |
| 🔄 Dual-channel parsing | AI replies carry both scene JSON and dialogue text in a single response |
| 📜 History & scene recall | Last 5 scenes saved, click to restore any previous layout |
| ✅ Validation system | AABB overlap detection, wall-penetration correction, desk–chair pairing check |
| 🖱️ Click-to-select | Click any object to prefill the NPC input: "Move the Desk …" |
| 🌐 Zero-cost deployment | WebGL build on itch.io, Vercel serverless proxy, no server to maintain |

---

## Tech Stack

```
Unity 2022.3 LTS  ──►  WebGL Build  ──►  itch.io
     │
     │  UnityWebRequest (POST JSON)
     ▼
Vercel Serverless (Node.js)          ← ZHIPU_API_KEY stored here, never exposed
     │
     │  REST API
     ▼
Zhipu GLM-4-Flash
```

| Layer | Technology |
|---|---|
| Engine | Unity 2022.3 LTS, Built-in Render Pipeline |
| AI model | Zhipu GLM-4-Flash |
| Backend proxy | Vercel Serverless (Node.js) |
| Avatar | VRoid Studio → UniVRM → Mixamo animations |
| Hosting | itch.io (WebGL) + GitHub |

---

## Project Structure

```
Assets/
├── Scripts/
│   ├── AIClient.cs          # HTTP layer — Ask() and AskNPC()
│   ├── SceneGenerator.cs    # JSON diff, incremental spawn, validation hook
│   ├── NPCController.cs     # Dialogue, alias resolution, dual-channel parse
│   ├── SceneValidator.cs    # AABB wall / overlap / pairing checks
│   ├── ObjectHighlighter.cs # Raycast click → prefill NPC input
│   ├── UIManager.cs
│   └── CameraController.cs
├── Resources/
│   └── SceneObjects/        # Desk, Chair, Sofa, Monitor … (Quaternius CC0)
├── Models/                  # FBX source files
├── Characters/              # Avatar.vrm + UniVRM generated assets
├── Animations/
│   ├── Clips/               # Idle.anim, Talking.anim
│   └── NPCAnimator.controller
└── Scenes/
    └── MainScene.unity

sceneforge-proxy/            # Vercel backend
├── api/
│   └── chat.js              # Rate limiting + GLM-4-Flash relay
└── vercel.json
```

---

## Running Locally

### Play (WebGL)
Open the itch.io page — no install needed.

### Run the Unity project

1. Unity 2022.3 LTS required
2. Install **UniVRM 0.x** from the [UniVRM releases](https://github.com/vrm-c/UniVRM/releases)
3. Clone this repo and open in Unity
4. The proxy is already live at `https://sceneforge-proxy.vercel.app/api/chat` — no API key setup needed to run

### Deploy your own proxy (optional)

```bash
cd sceneforge-proxy
npm install
# Create .env.local
echo "ZHIPU_API_KEY=your_key_here" > .env.local
vercel dev        # local
vercel --prod     # deploy
```

Then update `AIClient.cs`:
```csharp
private const string PROXY_URL = "https://your-proxy.vercel.app/api/chat";
```

---

## Available Prefabs

`Desk` · `Chair` · `Sofa` · `Table` · `Monitor` · `Bookcase` · `Whiteboard` · `Lamp` · `Plant` · `Projector` · `Door` · `Window`

All furniture from [Quaternius Furniture Pack](https://quaternius.com) — CC0 license.

---

## How Scene Generation Works

```
User types: "a library with reading corner"
        │
        ▼
chat.js injects SCENE_SYSTEM_PROMPT + user message
        │
        ▼
GLM-4-Flash returns pure JSON:
{"objects":[{"prefab":"Bookcase","x":-4.5,"y":0,"z":-2.5,...}, ...]}
        │
        ▼
SceneGenerator.OnAIResponse() extracts JSON, validates, diffs against
current scene, only spawns/destroys changed objects
        │
        ▼
SceneValidator checks: wall penetration → auto-correct position
                        AABB overlap    → push objects apart
                        desk–chair pair → warn if missing
```

---

## NPC Scene Editing

Walk within 2.5 units of the VRoid avatar → dialogue panel opens.

**Example commands:**
- `"Move the Desk to the left wall"`
- `"Rotate the Sofa 90 degrees"`
- `"Remove the Plant"`
- `"Add a Whiteboard near the door"`

The NPC system:
1. Resolves aliases (`couch` → `Sofa`, `shelf` → `Bookcase`, etc.)
2. Checks scene inventory client-side before calling AI
3. Sends full scene JSON as context with every request
4. AI returns `SCENE_UPDATE|||{json}|||reply text` — parsed in `NPCController`

---

## Security

- API key lives in Vercel environment variables — never reaches the browser
- Rate limiting: 20 requests / IP / minute (in-memory, resets on cold start)
- `Access-Control-Allow-Origin: *` is intentional for WebGL; the proxy is the security boundary

---

## License

Code: [MIT](LICENSE)  
Furniture assets: [CC0](https://quaternius.com)  
Avatar: created with VRoid Studio (personal use)