# ProtoPirate — Notes de projet

## Objectif
Activer la fonctionnalité TX (émulation/retransmission) dans ProtoPirate pour Flipper Zero sous firmware **Momentum**.

---

## Repo utilisé
**RocketGod-git/ProtoPirate**
https://github.com/RocketGod-git/ProtoPirate

> Note : Le repo officiel désactive volontairement le TX par défaut pour éviter la désynchronisation accidentelle des clés de voiture. Il faut patcher manuellement.

---

## Modifications appliquées

### 1. `defines.h` — Activer la feature TX
```c
// Avant :
// #define ENABLE_EMULATE_FEATURE

// Après :
#define ENABLE_EMULATE_FEATURE
```

### 2. `application.fam` — Permissions et stack size
```python
# Avant :
requires=["gui"],
stack_size=2 * 1024,

# Après :
requires=["gui","subghz"],
stack_size=8 * 1024,
```

**Pourquoi `stack_size=8 * 1024` ?**
Le Flipper Zero a 190KB de heap total. Avec toutes les features TX activées, il ne reste que ~15KB libre avec le stack par défaut de 2KB. Augmenter à 8KB évite le crash "Out of Memory" au lancement. Passer à `10 * 1024` si le crash persiste.

**Pourquoi `subghz` dans `requires` ?**
Nécessaire pour que le firmware Momentum autorise l'accès au stack Sub-GHz en mode TX.

---

## Compilation

### Pour Momentum (firmware actuel)
```bash
ufbt update --index-url https://get.momentum-fw.com/directory.json --channel dev
ufbt
```

### Pour Unleashed
```bash
ufbt update --index-url https://up.unleashedflip.com/directory.json --channel dev
ufbt
```

> ⚠️ Le canal de compilation **doit** correspondre au firmware installé sur le Flipper, sinon crash au démarrage.

Le `.fap` compilé se trouve dans `./dist/`.

---

## Problèmes rencontrés

| Problème | Cause | Solution |
|---|---|---|
| "Not enough RAM" au lancement | `stack_size` trop petit (2KB) | Augmenter à `8 * 1024` |
| TX ne fonctionne pas | `ENABLE_EMULATE_FEATURE` commenté | Décommenter dans `defines.h` |
| Crash sur Receive | Même cause que RAM | `stack_size=8 * 1024` ou `10 * 1024` |
| App incompatible | SDK mismatch firmware/compilation | Recompiler avec le bon `--index-url` |

---

## Références Reddit
- [Script bash piratebuild.sh (t4c)](https://www.reddit.com/r/flipperhacks/comments/1rxa3cx/simple_linux_bashscript_to_enable_emulation_in/)
- [Protopirate Crash Out of Memory](https://www.reddit.com/r/flipperhacks/comments/1qmaq67/protopirate_crash_out_of_memory/)
- [Emulating captured signals with Protopirate](https://www.reddit.com/r/flipperhacks/comments/1qivtvi/emulating_captured_signals_with_protopirate/)

---

## Contexte mémoire Flipper Zero
```
Heap total : 190 000 bytes
Heap libre avec toutes features ON : ~14 944 bytes  ← trop juste
Heap libre sans SUB_DECODE et TIMING_TUNER : ~28 192 bytes  ← OK
```
`ENABLE_TIMING_TUNER_SCENE` et `ENABLE_SUB_DECODE_SCENE` restent **commentés** dans `defines.h` pour économiser la RAM.
