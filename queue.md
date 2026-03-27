Increase softcap from 15 to 30 (less logit compression for small model)
Increase warmdown from 0.5 to 0.7 (longer cooldown since 0.5 beat 0.3)
Increase EMBEDDING_LR from 0.6 to 1.0 (higher embedding LR with more steps)
Set FINAL_LR_FRAC to 0.1 (don't decay LR all the way to zero)
