# Phase C Summary

## Scope
Phase C answered one model-side question on the current synthetic protocol: how `gating` and `regime` signals behave after entering `iTransformer`, under fixed split, seed, train budget, and evaluation rules.

## Fixed Protocol
- Dataset: synthetic `X.npy`
- Split: `phaseC_round1_split.json`
- Runtime: `itr` + GPU
- Seed: `2026`
- Evaluation unit: prediction elements without timestamp deduplication

## Comparison Table

| Variant | Config | Global (MSE/MAE) | Pre (MSE/MAE) | Post (MSE/MAE) | Switch Window (MSE/MAE) | Switch Pre (MSE/MAE) | Switch Post (MSE/MAE) | Takeaway |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| baseline | no signal | 0.8902 / 0.7155 | 0.9531 / 0.6916 | 0.8273 / 0.7395 | 0.6823 / 0.5999 | 0.3523 / 0.4319 | 1.0123 / 0.7679 | Reference point |
| gating-only best | `loss_weighting + direct + alpha=0.25` | 0.8867 / 0.7141 | 0.9505 / 0.6884 | 0.8230 / 0.7399 | 0.6860 / 0.6015 | 0.3550 / 0.4334 | 1.0170 / 0.7697 | Small global/non-switch gain, negative switch-window trade-off |
| regime-only light_aux_input | `light_aux_input`, gating off | 0.9016 / 0.7268 | 0.9445 / 0.6910 | 0.8586 / 0.7626 | 0.6770 / 0.5970 | 0.3492 / 0.4298 | 1.0049 / 0.7643 | Positive switch-window recovery, negative global/post trade-off |
| minimal gating+regime | `gating best + regime extra_time_feature` | 0.8864 / 0.7133 | 0.9521 / 0.6894 | 0.8208 / 0.7372 | 0.6880 / 0.6020 | 0.3553 / 0.4333 | 1.0207 / 0.7706 | Better global, but no switch-window recovery |
| gating+regime light_aux_input | `gating best + regime light_aux_input` | 0.9047 / 0.7289 | 0.9402 / 0.6876 | 0.8692 / 0.7702 | 0.6802 / 0.5984 | 0.3511 / 0.4309 | 1.0093 / 0.7660 | Recovers switch-window above baseline/gating-only, but with larger global/post trade-off |

## Phase C Conclusions
1. `gating-only` is a valid model-side signal for improving global/non-switch behavior, but it harms switch-window performance.
2. `regime-only light_aux_input` is the first regime path that improves `switch_window / switch_pre / switch_post`, but it worsens global and post-eval metrics.
3. `gating + regime light_aux_input` proves that regime's switch-window information can survive inside the joint model, but the current minimal fusion does not preserve gating's global advantage.
4. Therefore, Phase C does **not** yet produce a clean dual-win route that simultaneously keeps global gains and switch-window gains.

## Final Closeout Statement
Phase C is closed with a clear division-of-labor finding:
- `gating` behaves like a global/non-switch quality regulator.
- `regime` behaves like a switch-window recovery signal.
- The current minimal joint fusion cannot yet combine both advantages without a stronger global/post trade-off.

## Phase D Entry Question
If Phase D continues, the first mechanism question should be:

Can a more scoped joint design preserve `gating`'s global advantage while localizing `regime`'s effect so that switch-window gains remain but the global/post trade-off shrinks?
