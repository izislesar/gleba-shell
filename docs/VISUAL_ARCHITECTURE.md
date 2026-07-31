# Визуальная архитектура fork

Документ фиксирует визуальный слой ashell для этого персонального Hyprland rice
fork. Он описывает текущую реализацию и её границу: сохранять поведение модулей
и сервисов, меняя представление только там, где это возможно.

## UI-слой

UI представляет собой дерево Iced `view`, собираемое в следующих местах:

- `src/app.rs` — `App::view` creates the status-bar surface. It obtains the
  three sections, puts them in `Centerbox`, applies the bar background, border
  radius, height and padding.
- `src/modules/mod.rs` — maps configured module names to their `view` methods;
  builds the left, centre and right rows; wraps standalone modules and module
  groups.
- `src/modules/` — visual content for every bar module and its menus.
- `src/components/` — shared visual primitives. This repository currently
  keeps custom widgets here rather than in a `src/widgets/` directory.
- `src/theme.rs` — visual design tokens, Iced palette construction and shared
  widget styles.

Поток визуального слоя:

```text
config.toml -> Config / Appearance -> AshellTheme
                                      |
App::view -> Centerbox(left, centre, right) -> modules_section
                                             -> module view + shared components
```

`src/outputs.rs` владеет геометрией Wayland layer-surface вокруг этого дерева.
Это не обычный UI-компонент, хотя он использует margin, высоту и scale factor
панели.

## Владение визуальными аспектами

| Аспект | Основное расположение | Примечание |
| --- | --- | --- |
| Тема и общие стили | `src/theme.rs` | `AshellTheme`, создание палитры, шкалы spacing/radius/font-size, стили кнопок и workspace. |
| Схема внешнего вида для пользователя | `src/config.rs` (`Appearance`, `BarAppearance`, `MenuAppearance`) | Разбор TOML, значения по умолчанию и проверка palette, opacity, radius, margin, шрифта и масштаба. |
| Цвета | defaults в `src/config.rs`; mapping в `src/theme.rs` | `background_color`, `primary_color`, `success_color`, `warning_color`, `danger_color`, `text_color`, цвета workspace. |
| Размеры | `src/theme.rs` | `FontSize` содержит именованные размеры текста; `Space` — общую шкалу пикселей. Базовая высота панели — `HEIGHT` в `src/main.rs`. |
| Отступы и padding | `src/theme.rs`, затем отдельные UI-файлы | Общие tokens — `Space::{xxs..xxl}`. Общий padding модулей — `src/components/module_item.rs`; меню/модули добавляют локальные отступы. |
| Радиусы | `src/theme.rs`; radius панели — `src/config.rs` | Общая шкала `Radius::{sm, md, lg, xl}`. `[appearance.bar].radius` выбирает значение для каждого угла. |
| Прозрачность | `src/config.rs`, `src/app.rs`, `src/components/module_group.rs`, `src/components/menu.rs` | Opacity панели — `[appearance.bar].opacity`; opacity/backdrop меню — `[appearance.menu]`. Режим `transparent` делает каждую группу модулей island. |
| Layout панели | `src/app.rs`, `src/modules/mod.rs`, `src/components/centerbox.rs` | Панель — трёхслотовый `Centerbox`; модули приходят из `[modules].left`, `.center`, `.right`. |
| Группировка модулей | `src/modules/mod.rs`, `src/components/module_group.rs`, `src/components/module_item.rs` | В прозрачном режиме группа разделяет один island; элементы имеют общие padding, hover и pointer handling. |
| Меню | `src/components/menu.rs`, `src/components/menu_wrapper.rs`, `src/components/sub_menu_wrapper.rs` | Общая рамка, размещение, backdrop и анимация открытия/закрытия. |
| Иконки | `src/components/icons.rs` | Static icon-to-glyph mapping и icon buttons. Иконки приложений workspace/tray загружаются через `src/services/xdg_icons.rs`. |
| Шрифты | `src/main.rs`, `src/components/icons.rs`, `build.rs`, `assets/` | `appearance.font_name` выбирает системный текстовый шрифт. Nerd Font subsets и custom icon font встроены; `build.rs` строит subset из glyph, используемых в `icons.rs`. |

### Важные фиксированные значения

- `src/theme.rs`: spacing tokens are 4, 8, 12, 16, 24, 32 and 48 logical px;
  radius tokens are 4, 8, 16 and 32 px.
- `src/main.rs`: base bar height is `34` logical px. `scale_factor` from
  `[appearance]` scales the layer surface and the UI.
- Individual components may intentionally use fixed dimensions, for example
  quick-setting rows in `src/components/quick_setting_button.rs`.

## Карта компонентов

### Верхняя панель

- `src/app.rs` — top-level bar view, surface background, bar radius, panel
  height and outer padding.
- `src/modules/mod.rs` — section/module composition and module interaction
  wrapping.
- `src/components/centerbox.rs` — custom three-section layout; keeps the
  centre section visually centred and optionally animates its x-position.
- `src/components/module_group.rs` and `src/components/module_item.rs` —
  transparent-mode islands, common item padding and hover behaviour.

### Индикатор workspace

- `src/modules/workspaces.rs` — workspace-button contents, dimensions,
  internal icon layout and state selection.
- `src/theme.rs` — `workspace_button_style`, including active, empty, urgent
  and per-workspace colour styling.
- `src/config.rs` — `WorkspacesModuleConfig`, including names, indicator
  format and workspace colour lists through `Appearance`.

### Системные модули

- `src/modules/system_info.rs` — CPU, memory, temperature, disk and network
  presentation, plus its detail menu.
- `src/modules/settings/mod.rs` — settings indicator strip and quick-settings
  menu composition.
- `src/modules/settings/audio.rs`, `network.rs`, `bluetooth.rs`, `power.rs`,
  `brightness.rs` — the corresponding settings cards, sliders and submenus.
- Other bar module views are neighbouring files in `src/modules/`, including
  `tempo/`, `media_player.rs`, `notifications.rs`, `privacy.rs`,
  `keyboard_layout.rs`, `keyboard_submap.rs`, `window_title.rs`, `updates.rs`
  and `custom_module.rs`.
- Shared primitives for these modules are in `src/components/`, notably
  `quick_setting_button.rs`, `slider_control.rs`, `format_indicator.rs`,
  `divider`, `button.rs` and `sub_menu_wrapper.rs`.

### Tray

- `src/modules/tray.rs` — bar tray-icon layout and tray menu view.
- `src/services/tray/` — D-Bus/status-notifier protocol integration. Treat it
  as backend code, not as a visual customization point.
- `src/services/xdg_icons.rs` — application icon loading for tray and optional
  workspace app icons. It is data loading, not the static icon design system.

### Меню Settings

- `src/modules/settings/mod.rs` — menu shell, quick-settings composition and
  navigation between submenus.
- `src/modules/settings/{audio,network,bluetooth,power,brightness}.rs` — each
  subpanel's view.
- `src/components/menu.rs`, `menu_wrapper.rs` and `sub_menu_wrapper.rs` —
  shared frame, positioning, visual surface and animation.
- `src/theme.rs` — quick-settings, submenu, button and module hover styles.

## Что можно менять на каждом уровне

### Только конфигурация — предпочтительно

Используйте пользовательский `~/.config/ashell/config.toml` (либо путь,
переданный через `--config-path`). Изменение репозитория не требуется для:

- palette and contrast: all `[appearance]` colours and workspace colours;
- system text font: `appearance.font_name` (requires restart);
- global UI/layer scale: `appearance.scale_factor`;
- bar form: `[appearance.bar]` `surface`, `opacity`, `radius`, `margin`;
- menu opacity and backdrop: `[appearance.menu]`;
- module placement and island grouping: `[modules]` `left`, `center`, `right`;
- module content formats, visibility and ordering where each module's existing
  TOML options expose them (for example workspace names/formats, clock format,
  system-info indicators and settings indicator formats);
- enabled/disabled built-in animations: `[animations].enabled`.

Это путь с наименьшим расхождением с upstream, его стоит пробовать первым.

### UI-код — допустимо для визуального fork

Use targeted Rust changes when configuration cannot express the desired visual
result, while retaining existing messages and service calls:

- `src/theme.rs` for token values, colour derivation, shared state styles and
  component radii;
- `src/app.rs` for the panel's visual container/padding only;
- `src/components/{module_group,module_item,button,menu,sub_menu_wrapper,quick_setting_button,slider_control,format_indicator}.rs`
  for shared presentation;
- the specific `src/modules/*.rs` view that owns a module being restyled;
- `src/components/icons.rs` for a deliberate icon-glyph swap, with awareness
  of the generated-font dependency described below.

Prefer a change in the narrowest owning module. Put a repeated styling rule in
`theme.rs` or a shared component only when multiple modules genuinely share it.

### Rust-логика — избегать, если визуальное требование не решается иначе

These changes alter behaviour, lifecycle or configuration contracts rather than
only presentation:

- `src/config.rs` when adding or changing TOML fields, defaults or validation;
- `src/app.rs` / `src/modules/mod.rs` when changing module dispatch, actions or
  menu-routing semantics;
- `src/outputs.rs` and layer-shell calls in `src/main.rs` when changing output
  creation, exclusive zones, anchors or surface lifecycle;
- `src/components/{centerbox,menu_wrapper,position_button,collapsible,animated_size}.rs`
  when changing custom widget layout, input or animation mechanics;
- every file under `src/services/`, plus compositor and IPC code.

For this fork's scope, do not make these changes for aesthetics alone. In
particular, do not surface raw hardware names in `system_info`; present useful
state such as utilization, temperature or battery instead.

## Безопасные файлы для визуальных правок

These are safe in the sense that they are presentation-oriented; keep changes
small and verify with `make check`:

- `src/theme.rs`;
- `src/components/button.rs`, `module_group.rs`, `module_item.rs`,
  `quick_setting_button.rs`, `slider_control.rs`, `format_indicator.rs`,
  `sub_menu_wrapper.rs` and `spinning_icon.rs`;
- `src/components/menu.rs` only for menu surface styling, not its lifecycle;
- the individual target module view in `src/modules/` (for example
  `workspaces.rs`, `system_info.rs`, `tray.rs`, or a file under
  `src/modules/settings/`);
- `src/components/icons.rs` for a focused glyph mapping change;
- the active user configuration file for palette/layout-only experiments;
- `docs/` for fork-specific design notes.

`src/app.rs` is conditionally safe for narrow bar-container changes. Preserve
the `Centerbox` composition, menu-close interaction and output IDs.

## Файлы, которые лучше не менять ради совместимости с upstream

- `src/services/**` — system integrations, D-Bus clients and backend state.
- `src/services/compositor/**` — Hyprland/Niri/generic compositor abstraction.
- `src/ipc.rs`, `src/xdg.rs`, `src/i18n.rs` — external integration and shared
  application infrastructure.
- `src/outputs.rs` — output discovery, Wayland surfaces, anchors, margins and
  exclusive-zone lifecycle.
- Most of `src/main.rs` — startup, CLI and font registration. The `HEIGHT`
  constant is visual but affects layer geometry, so change it only with a
  deliberate sizing decision and multi-output testing.
- `src/config.rs` — keep the upstream configuration schema stable; use existing
  options rather than adding rice-specific configuration knobs.
- `src/components/centerbox.rs`, `menu_wrapper.rs`, `position_button.rs`,
  `collapsible.rs`, `animated_size.rs` — custom widget behaviour is reusable
  infrastructure and easy to regress through cosmetic edits.
- `build.rs`, `assets/SymbolsNerdFont-*.ttf`,
  `assets/AshellCustomIcon-Regular.otf` — the build script derives the embedded
  Nerd Font subsets from `\\u{...}` glyphs in `src/components/icons.rs`. Do not
  manually edit generated files in `target/generated/`.
- `website/versioned_docs/` — frozen upstream documentation snapshots.

## Рекомендуемый порядок визуальных изменений

1. Prototype colour, opacity, font, bar mode, margins and module layout in
   `config.toml`.
2. If a shared visual token is missing, adjust `src/theme.rs` or one shared
   component.
3. Restyle only the owning module view for a module-specific treatment.
4. Keep changes separate by concern (for example theme, workspaces, panel) and
   run `make check` before committing.
