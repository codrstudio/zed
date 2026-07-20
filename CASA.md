# CASA — rebrand do fork Zed → app da casa (Kai)

Fork nosso: `codrstudio/zed` (GPL-3.0). Objetivo: rebrandar + pré-configurar pra família abrir e já falar com a Kai, zero config. Ancora: frente `mind/effort/on/kai-viva`.

Mapa de coordenadas de edição (recon 2026-07-19). GPL: nosso fork fica aberto; **não** usar marca/logo "Zed" no que distribuir.

## PONTO 2 — Embutir a Kai (o mais fácil e o mais importante)

**`assets/settings/default.json:2754`** — hoje `"agent_servers": {}`. Trocar por:
```json
"agent_servers": {
  "Kai": { "type": "custom", "command": "node", "args": ["<caminho>/acp-agent.mjs"], "env": {} }
}
```
Isso embute o agente no binário — usuário não edita nada. Pipeline `reregister_agents` já instancia `Custom`. **Zero Rust novo.**
- Schema: `crates/settings_content/src/agent.rs:719-780` (`CustomAgentServerSettings::Custom`)
- Runtime: `crates/project/src/agent_server_store.rs:294-434`
- Cuidado: se o user definir `agent_servers` próprio, o merge por chave preserva "Kai" a menos que ele redefina a chave.

## PONTO 3 — Tirar DeepSeek e default pro Kai

- **Remover DeepSeek (fácil, isolado):** apagar `crates/language_models/src/language_models.rs:271-278` (register) + import `:9`. Opcional: dep em `crates/language_models/Cargo.toml`, crate `crates/deepseek/`, ícone `assets/icons/ai_deep_seek.svg`.
- **Zed Agent nativo (médio — ocultar, NÃO arrancar):** `ZED_AGENT_ID` em `crates/agent/src/agent.rs:2562`; `enum Agent`/`Agent::NativeAgent` em `crates/agent_ui/src/agent_ui.rs:438-497` (label "Zed Agent" `:467`). Trocar `selected_agent` default de `Agent::NativeAgent` pro nosso `Custom("Kai")` em `crates/agent_ui/src/agent_panel.rs` (`:10742-10752`, `:11580`); lista do menu em `:5793`/`:6085-6102`. Arrancar a variante do enum quebra ~20 refs + testes — **não recomendado**; ocultar + trocar default é cirúrgico.

## PONTO 1 — Nome e marca

- **Nome (ponto único mais crítico):** `crates/release_channel/src/lib.rs:192-199` `display_name()` → "Zed"/"Zed Dev"/... Troca aqui muda janela/menus.
- **App/bundle IDs:** `release_channel/src/lib.rs:214-221` `app_id()` (`dev.zed.Zed`...); `crates/zed/Cargo.toml:278-308` (`[package.metadata.bundle-*]`: identifier, name, icon, `osx_url_schemes`).
- **About:** título `crates/zed/src/zed.rs:1662-1663`; ícone embutido `:1472-1485`; menu `crates/zed/src/zed/app_menus.rs:66`; URLs `zed.dev` `:305-319`.
- **Assets (trocar arquivo, manter nome = fácil):** `crates/zed/resources/app-icon.png` (+@2x, `-dev/-nightly/-preview`); Windows `crates/zed/resources/windows/app-icon.ico`; logos in-app `assets/icons/zed_agent*.svg`, `zed_assistant.svg`, `ai_zed.svg` (enum `IconName`).
- **Instalador Windows (Inno):** `crates/zed/resources/windows/zed.iss` (`AppPublisher` `:5`, URLs `:6-8`, `SetupIconFile` `:21`). `AppName`/`AppId` vêm de `-D` no CI (`script/` `ISCC /D`), não hardcoded.
- **Linux:** `resources/zed.desktop.in`, `flatpak/zed.metainfo.xml.in`, `snap/snapcraft.yaml.in`.

## Identidade visual — claude-base + marca KeepCoding

**Casamento (decidido com o Guga, 2026-07-19):** a base estética é o **tema claude** (neutros quentes, sensação de papel — o CSS que o Guga passou, OKLCH), e a **identidade** vem da **marca KeepCoding** (`workspace/kai/marca`, submódulo `keepcoding-ai/marca`).

- **Base/canvas:** paleta claude quente (light+dark, do CSS colado). Backgrounds, texto, muted, surfaces.
- **Acento/identidade:** **laranja KeepCoding `#e35336`** (cor-mãe) no lugar do primary clay do claude → é o que faz virar NOSSO. Coral `#f0a89a` = acentos suaves. Tinta `#161213` (quase-preto quente) pro texto escuro.
- **Ícone do app:** `marca/brand/{light,dark}/icon.svg` — o `k` entre brackets `‹ ›` no squircle laranja. Vira o app-icon (png/ico) do Zed + o ícone do About.
- **Logo (splash/About):** `marca/brand/{light,dark}/logo.svg` (símbolo+wordmark) ou `logo-v`/`logo-h` conforme o encaixe.
- **Fonte de marca:** **Philosopher** (serif, em `marca/Philosopher.zip`) — voz da marca, títulos/assinatura. UI/editor seguem DM Sans / Menlo do tema claude.
- **Favicon/wallpaper/texture/avatar:** prontos na raiz de `marca/`.

**Regras da marca (do README):** o laranja manda; não distorcer/recolorir/reorganizar o símbolo (k+brackets+squircle é fixo); SVG preferível; margem = altura do `k`.

**Tradução pro Zed (tarefa real):** o CSS colado é Tailwind/web — o Zed usa **tema próprio em JSON** (`assets/themes/*.json`, schema com `style`: background/text/element/editor/syntax/players). Não dá pra colar o CSS; eu **construo um tema Zed** (`assets/themes/keepcoding.json`, light+dark) a partir desses OKLCH + injetando o laranja `#e35336` nos acentos, e seto como default no `default.json` (`"theme": "KeepCoding"`).

**Nome do app: `KeepCodium`** (decidido Guga, 2026-07-19). Vai em `display_name()` (`release_channel/lib.rs:192-199`), `app_id()` (`dev.keepcoding.KeepCodium`?), bundle metadata (`Cargo.toml:278-308`), About (`zed.rs:1662`), menu (`app_menus.rs:66`), instalador (`zed.iss` via `-D AppName`). NOTA (corrigida pelo Guga, 2026-07-19): existe um `keepcodium` do Forge (`D:/forge/workspace/forge/keepcodium`, receita VSCodium portátil) que só **coincide no nome** — o nosso NÃO é evolução daquele. Projeto próprio, desenho próprio (fork Zed + Kai via ACP). Ignorar o do Forge; não absorver nada de lá.

## Ordem sugerida (barato→caro, visível primeiro)

1. **`default.json:2754`** embute Kai → app já abre falando comigo (a prova do "experiência pronta"). Zero Rust.
2. **Remover DeepSeek** + default pro Kai → some a confusão do agente nativo.
3. **Nome** em `release_channel/src/lib.rs` + About → vira "nossa marca".
4. **Assets** (logo/ícone/splash) → a cara.
5. **Instalador/CI** (`-D AppName`, assinatura) → distribuível.

Passos 1-2 provam o conceito local numa sessão. 3-5 são o acabamento de produto (frente-filha / harness).
