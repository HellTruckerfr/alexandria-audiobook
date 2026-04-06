<img width="475" height="467" alt="Alexandria Logo" src="https://github.com/user-attachments/assets/fa2c36d3-a5f3-49ab-9dfe-30933359dfbd" />

# Alexandria Audiobook Generator — Fork HellTruckerfr

> Fork personnalisé d'Alexandria optimisé pour la génération d'audiobooks en français à grande échelle, avec pipeline batch GPU sur serveur distant (vast.ai RTX 4090), voix LoRA françaises, et détection automatique des personnages.

Repo original : [Finrandojin/alexandria-audiobook](https://github.com/Finrandojin/alexandria-audiobook)

---

## Ce que ce fork ajoute

### Pipeline Batch GPU (External LoRA Mode)
- **Batch GPU via serveur distant** — Alexandria envoie les chunks par lots de 6 au serveur Gradio distant au lieu de les générer un par un
- **Sous-batching automatique** — découpe les gros chapitres en sous-batches pour éviter les OOM sur GPU
- **Nettoyage VRAM** — `torch.cuda.empty_cache()` entre chaque sous-batch
- **~8 minutes par chapitre** sur RTX 4090 vs 15-20 minutes en mode séquentiel
- **Fallback intelligent** — les chunks non-LoRA sont traités séquentiellement, les LoRA en batch

### Voix LoRA Françaises
- **narrator_fr_v3** — voix narrateur français entraînée sur 1697 samples, 8 epochs
- **Détection automatique de genre** — Claude Haiku détecte masculin/féminin pour les nouveaux personnages
- **Assignation automatique** — tout personnage inconnu reçoit narrator_fr_v3 automatiquement
- **Sauvegarde permanente** — les nouveaux personnages sont sauvegardés dans `voice_config.json`
- **Pré-détection avant TTS** — les nouveaux personnages sont détectés et enregistrés AVANT la génération

### Voix SYSTEM (Messages de jeu)
- Tout texte entre `[...]` est automatiquement assigné au personnage `SYSTEM`
- Instruct forcé : voix robotique et métallique
- Règle intégrée dans les prompts LLM et review

### Post-processing Audio
- **Variation de volume automatique** — cris/fureur +6dB, chuchotements -8dB, via analyse des instructs
- **Trim start 300ms** — supprime l'artefact "ur" en début de phrase côté serveur
- **Silence final 1s** — évite la coupure brusque de la dernière ligne

### Prompts Expressifs
- **Philosophie instruct** — décrit ce que la VOIX fait physiquement, pas l'émotion abstraite
- **Marqueurs prosodiques** — pace, breath, pitch, texture, volume
- **8 options narrator** au lieu de 4
- **SYSTEM messages** — détection automatique des `[...]`
- **Review prompt** — réécrit automatiquement tout instruct vague ou trop court

### Cache Scripts LLM
- Les scripts JSON générés sont sauvegardés dans `scripts_cache/Chapitre_XXXX.json`
- Si le cache existe → skip LLM + review, charge directement
- Relancer un chapitre ne coûte plus rien en tokens API

### Batch Pipeline Automatisé
- **Onglet Batch** dans l'interface web
- Traite automatiquement des dossiers entiers de chapitres
- Skip les chapitres déjà générés (détection par fichier MP3 existant)
- Retry automatique des chunks manquants (5 tentatives)
- Nettoyage voicelines/ avant ET après merge
- Silence 1s ajouté à la fin de chaque MP3

### Merge M4B avec Chapitrage
- Scripts PowerShell pour merger en fichiers M4B avec chapitres titrés
- Structure compatible Smart Audiobook Player (Android)
- Format : `JKSManga/My Vampire System/Arc X - Titre/fichier.m4b`

---

## Configuration requise

- [Pinokio](https://pinokio.computer/)
- **LLM** : Claude API (Anthropic) — `claude-haiku-4-5-20251001` recommandé
- **TTS** : Serveur Gradio distant (vast.ai RTX 4090) via tunnel SSH
  - Repo serveur : [HellTruckerfr/xtts-vllm-autoserver](https://github.com/HellTruckerfr/xtts-vllm-autoserver)
- **ffmpeg** avec libmp3lame : `C:/ffmpeg/bin/ffmpeg.exe`

---

## Configuration Alexandria

| Paramètre | Valeur |
|-----------|--------|
| TTS Mode | External Server |
| TTS Server URL | http://127.0.0.1:7860 |
| LLM Base URL | https://api.anthropic.com/v1 |
| LLM Model | claude-haiku-4-5-20251001 |
| Parallel Workers | 6 |
| Language | French |

---

## Voix LoRA disponibles

| Adapter | Usage | HuggingFace |
|---------|-------|-------------|
| narrator_fr_v3 | Tous personnages (masculins, féminins, inconnus) | `Helltrucker/audiobook-lora-models` |

---

## Tunnel SSH (vast.ai)

Lancer le watchdog de reconnexion automatique :
```powershell
powershell -ExecutionPolicy Bypass -File "tunnel_watchdog.ps1"
```

---

## Structure ajoutée

```
Alexandria/
├── scripts_cache/             # Scripts JSON mis en cache (skip LLM si existe)
├── voice_config.json          # Configuration des voix par personnage
├── tunnel_watchdog.ps1        # Auto-reconnect SSH tunnel vers vast.ai
└── titres chapitres/
    ├── merge_all_arcs.ps1     # Merge tous les arcs en M4B
    ├── merge_to_m4b.py        # Script de merge M4B avec chapitrage
    └── extract_titles.py      # Extraction des titres depuis fichiers d'arcs
```

---

## License

MIT — voir repo original [Finrandojin/alexandria-audiobook](https://github.com/Finrandojin/alexandria-audiobook)
