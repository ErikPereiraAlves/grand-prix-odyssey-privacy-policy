# Pinguim Odyssey

Jogo de plataforma vertical com pinguim em Flutter + Flame, focado em
publicação na **Google Play Store (Android only)**.

- Package: `com.alveser.pinguimodyssey`
- Min SDK: 21 / Target SDK: 36 / Compile SDK: 36
- AGP 8.10.0 · Gradle 8.11.1 · Kotlin 2.1.0 · JDK 17

## Como rodar

```bash
flutter clean
flutter pub get
flutter run
```

Para gerar o bundle de release para a Play Store:

```bash
flutter build appbundle --release
```

> Antes de publicar, troque `signingConfig signingConfigs.debug` em
> `android/app/build.gradle` por uma `release` configuration real, e troque
> os IDs de teste do AdMob em `lib/services/ad_service.dart` e o
> `APPLICATION_ID` em `AndroidManifest.xml` pelos seus IDs reais.

## Stack

| Item | Versão |
|---|---|
| Flutter SDK | >= 3.2.0 |
| Flame | ^1.18.0 |
| flame_audio | ^2.10.0 |
| google_mobile_ads | ^5.1.0 |
| video_player | ^2.9.1 |
| shared_preferences | ^2.3.0 |
| intl | ^0.20.2 |

## Estrutura

```
lib/
├── main.dart
├── game/
│   ├── penguin_game.dart      # FlameGame + loop principal
│   └── components/            # Pinguim, plataformas, obstáculos, items, HUD
├── models/                    # GameState (persistência), LevelConfig (10 fases)
├── screens/                   # Menu, level select, game, settings, vídeo
├── services/                  # AudioService, AdService, StorageService
└── l10n/                      # 12 idiomas (en/pt/es/fr/de/it/ja/ko/zh/ru/ar/hi)

assets/
├── images/{penguin,obstacles,items,backgrounds,ui}/
├── audio/{music,sfx}/
└── videos/                    # intro.mp4, end.mp4
```

Os componentes de jogo desenham placeholders com `Canvas` quando os PNGs/MP3
ainda não existem — substitua os arquivos em `assets/` para ver os sprites
e sons reais.

## Changelog desta revisão

### Bugs críticos consertados
- Input do jogo não funcionava: `TapCallbacks`/`DragCallbacks` em `FlameGame`
  era inválido. Substituído por `GestureDetector` envolvendo o `GameWidget`
  em `game_screen.dart`, que chama `game.handleTap()` / `game.handleDrag()`.
- Pouso em plataforma quebrado: `_isOnPlatform` era resetado todo frame, o
  que fazia o pinguim cair mesmo em cima da plataforma. Reescrito usando
  `onCollision` (todo frame) com snap para o topo.
- Comparação de colisão considerava o centro do pinguim vs o topo da
  plataforma; agora usa o pé do pinguim com tolerância.
- Score era penalizado pelo speed boost (que diminui scrollSpeed).
  Score agora cresce com a velocidade base, não a atual.

### Correções de qualidade
- `intl` agora é `^0.20.2` (antes estava pinado em `0.20.2` sem caret).
- Flame atualizado para `^1.18.0`, flame_audio para `^2.10.0`,
  video_player `^2.9.1`, shared_preferences `^2.3.0`.
- Substituído `HasGameRef` (deprecated) por `HasGameReference` e
  `gameRef.x` por `game.x` em todos os componentes.
- Substituído `withOpacity()` (deprecated no Flutter recente) por
  `withValues(alpha: …)` em todo o código.
- `print()` substituído por `debugPrint()` em todo o código; lint
  `avoid_print` reativado em `analysis_options.yaml`.
- `clamp(1, 10)` hardcoded substituído por `LevelConfig.levels.length`.
- `StorageService.getInstance()` agora é seguro contra inicialização
  concorrente.
- `playSfx` agora é `async` e propaga erros corretamente.
- `VideoScreen` agora tem proteção contra duplo callback de finalização.
- `_spawnInitialPlatforms` reescrito para evitar sobreposições.
- `_jump()` agora só funciona quando há plataforma sob o pinguim
  (sem mais "double jump infinito").
- Removido o campo morto `_elapsedTime`.

### Mudanças Android
- Migrado para AGP 8.10.0 / Gradle 8.11.1 / Kotlin 2.1.0 / JDK 17.
- Removido todo o suporte a iOS no código Dart (`AdService` agora é
  Android only).
- Package atualizado para `com.alveser.pinguimodyssey`:
  - `namespace` e `applicationId` em `app/build.gradle`
  - Caminho do pacote Kotlin: `kotlin/com/alveser/pinguimodyssey/MainActivity.kt`
  - Pacote dentro do `MainActivity.kt`
  - `AndroidManifest.xml` (removido atributo `package`, que foi substituído
    por `namespace` no AGP 8)
- `proguard-rules.pro` expandido (Flame audio, media3, embedding 2).
- Adicionados recursos `values-night/styles.xml` e
  `drawable-v21/launch_background.xml` (padrão Flutter para evitar
  warnings em dispositivos modernos).
- `gradle.properties`: heap maior, `nonTransitiveRClass=true`.
- `shrinkResources true` no buildType release.
- Adicionado bloco `<queries>` no AndroidManifest (recomendação Flutter
  para Android 11+).

## Itens importantes antes de publicar

1. Substituir os IDs de teste do AdMob (`ad_service.dart` e `APPLICATION_ID`
   no `AndroidManifest.xml`) pelos IDs reais.
2. Configurar uma `signingConfig` de release real (atualmente está usando
   o keystore de debug, o que **não funciona** para upload na Play Store).
3. Adicionar os assets reais em `assets/images/`, `assets/audio/` e
   `assets/videos/`.
4. Adicionar ícones em `android/app/src/main/res/mipmap-*/ic_launcher.png`
   (use `flutter_launcher_icons` para gerar).
