# Laboratório Brasil: feature roadmap

The long-term plan for what the platform teaches. The [README](../README.md) covers the system; this file covers the product.

## Modules

| Module | What it covers | Lands in | Status |
|---|---|---|---|
| Dashboard | Daily mission, continue learning, vocabulary due, current course/book/podcast, study statistics | Portal | Later |
| Course library | UT Austin, FSI and COERLL material: units and lessons, grammar notes, exercises, vocabulary, personal notes, bookmarks, external links | fsi-scraper | Building |
| Vocabulary | Word or phrase, translation, definition, example sentence, audio, difficulty, frequency, category, lesson, tags, personal notes | Fichas | Planned |
| Grammar | Present tense, past tenses, future, subjunctive, pronouns, prepositions, passive voice, connectors, common Brazilian constructions | Grammar drills | v2 |
| Journal | Daily writing, corrections, new vocabulary, grammar learned, common mistakes, AI suggestions | Coach | Planned |
| Listening lab | Podcasts, audiobooks, radio, interviews, transcripts, slow playback, shadowing, listening quizzes, bookmarks | Livros | Planned |
| Speaking lab | Restaurant, airport, hotel, beach, doctor, office, coffee shop, bus, Uber, market, Rio street conversations. Feedback on grammar, vocabulary, pronunciation, fluency and naturalness | Whisper + Tutor | Later |
| AI tutor | Explain grammar, generate quizzes, create examples, identify recurring mistakes, adapt exercises, generate flashcards, conversation practice | Tutor | Running (basic) |
| Reading assistant | Open an article, click a word, definition, pronunciation, examples, grammar information, save vocabulary, generate flashcards | Reading tracker | Later |
| Brazil explorer | State, city, neighborhood, then culture, history, food, music and vocabulary | | Later |
| Carioca mode | Rio pronunciation, street expressions, beach, bus and bar conversations, everyday Rio vocabulary, local etiquette | Gíria | Planned |
| Culture | Music, literature, food, football, festivals, Indigenous cultures, Afro-Brazilian history and culture, architecture, Brazilian history | | Later |
| Music and movies | Artist, album and movie pages, vocabulary, expressions, cultural context | Música, Vídeo | Planned |
| Photography | Photograph, Portuguese caption, camera settings, location, vocabulary, cultural notes, story | | Later |
| Analytics | Hours studied, vocabulary learned, books completed, listening hours, writing streak, conversation time, grammar progress | Portal | Later |

## Content rule

Course material from UT Austin and FSI is stored as links, metadata and my own notes unless its license allows more. Nothing is republished wholesale.

## Infrastructure roadmap

| Item | Status |
|---|---|
| Fix Navidrome env vars, load music | Now |
| fsi-scraper + CloudNativePG | Now |
| Monorepo layout | Now |
| Sealed Secrets | Next |
| kube-prometheus-stack | Next |
| Harden every app | Next |
| Flux image automation | Next |
| Traefik hostnames, Cloudflare Tunnel | Next |
| Dev workflow for fsi-scraper and coach: mise, pre-commit, uv, Trivy, GitHub Actions, GHCR, k3d tests | Next |
| Ansible provisioning | Later |
| Terraform + AKS | Later |
| CloudNativePG backups and tested restore | Later |
| Shared storage | Later |
| Tailscale | Later |
| Loki, Uptime Kuma | Later |
| MinIO, FFmpeg, Whisper | Later |
| 2 TB drive on `dell-node` | Later |
| PR review with Claude Code, n8n review agent | Later |
| Omni, Talos, Cilium, External DNS | Second cluster on `biggie-smalls` |
| Forgejo | Later |
| Authentik SSO, search | v2 |
