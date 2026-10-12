# Workflow editor p7 — step 2: node appearance

Commit: agdevworld `5e38f75`.

## Changes

- **Model and file.** `nodes.<id>.appearance: {icon, color}` is optional (`shared/model.ts`). Icons and colors are fixed tokens: `APPEARANCE_ICONS` (`study do delegate talk present cleanup test doc build review flag`) and `APPEARANCE_COLORS` (`blue amber violet teal green rose slate orange`). The YAML shape (`server/yamlDoc.ts`) writes a new appearance in flow style next to its node, keeps the comment of an edited entry, and removes the key when it is cleared. An unchanged file is still written back byte for byte.
- **Validation.** An unknown icon or color produces the warning `appearance-icon` / `appearance-color`, and the type's default is drawn. A wrongly shaped value (e.g. a list) is a shape problem like any other field.
- **Digests and compilation.** Appearance is not in the definition or intent projection, nor in `authoredNode()` / the compiled graph. The projection kind and compiler version are unchanged (`ag.wfe-compile.v2`), so no existing approval or run is affected.
- **Presets.** `NODE_PRESETS` defines *Present* (`do`, `present`, `teal`) and *Clean up* (`do`, `cleanup`, `slate`), each with a starting description. The editor's palette has `+ Present` / `+ Clean up` beside the four types; they add ordinary `do` nodes whose name, description and appearance stay editable.
- **Inspector.** Each node has an *Appearance* field: an icon and a color select, each with "type default". It states that this is display only.
- **Canvas and run views.** The card uses the node's icon and a palette fill and border (`.color-<token>`), with a head band in the token color. Outgoing edges take the token color. The type label stays ("DO"). Text is forced dark on colored cards, so a colored delegate card stays readable. Run state is unchanged: the state pill keeps its white background, its own colored text and its glyph, and the state ring keeps its color. The run view draws the snapshot's YAML, so runs and history views show the appearance they were created with.

## Verification

- `test/appearance.test.ts` (4 tests): round trip, digest/approval/compile invariance, warnings, and that presets run like any `do`. Full suite `npm test`: **176/176 pass** (172 before p7 + 4). `tsc --noEmit` is clean; `npm run build` succeeds.
- `checks/appearance.ts` (browser, private service on :8198): 15/15 checks pass —
  - presets offered and added as `do` nodes;
  - inspector color applied;
  - saved YAML in flow style;
  - in the run view: the running node in teal still shows "✎ Running (reported)" on its own pill, the running ring keeps `rgb(111,179,214)`, and the pending cleanup node shows its slate look from the snapshot;
  - no console errors.
- Screenshots: `agdevworld/.local/wfe-checks/screenshots/p7-appearance-editor.png`, `p7-appearance-run.png` (ignored).

## Limitations

- Colors are a fixed palette, not free CSS. That is deliberate: it is what lets the editor guarantee state legibility.
- Only the card and its outgoing edges are restyled. The generated documentation steps keep their look.

## Next

Step 3: presentation records (`presentation.register`, `presentation.observe`), the frozen bundle, the review screen, and binding the confirmation question and decision to a presentation revision.
