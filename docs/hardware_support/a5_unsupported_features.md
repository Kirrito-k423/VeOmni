# A5 (Ascend 950) Unsupported Features and Limitations

This document lists features and scenarios that are **not supported** or have **limited support** on Ascend 950 (A5) products.

> For the full hardware support matrix, see [Get Started with Ascend NPU](get_started_npu.md).

## Unsupported Features on A5

| Feature / Scenario | Reason | Recommended Alternative |
|---|---|---|
| Multi-node Expert Parallelism (EP) for MoE models > 8B | A5 inter-chip bandwidth is lower than A2 (910B), multi-node EP for large MoE models is not validated | Use single-node EP or FSDP2 for MoE models on A5 |
| Wan2.1 (1.3B) training | Wan2.1 on NPU uses FSDP1 backend, which is not validated on A5 | Use A2 (910B) for Wan2.1 training |
| Training models > 8B parameters | A5 HBM capacity is limited; models larger than 8B may cause OOM | Use A2/A3 for models larger than 8B |
| Sequence length > 32K for models ≥ 30B | A5 memory bandwidth and capacity cannot support very long sequences on large models | Reduce sequence length or use A2/A3 |
| `flash_attention_2` with `triton` backend | The Triton-based flash attention backend is not available on A5 | Use `sdpa` or CANN built-in attention implementation on A5 |
| `liger_kernel` MoE backend | Liger-Kernel does not support A5 hardware | Use `moe_implementation: fused_npu` on A5 |
| Multi-node training with > 8 nodes | A5 cluster configurations have only been validated up to 8 nodes | Contact the VeOmni team for larger cluster support |

## NPU Operator Implementation Recommendations for A5

When running on A5, configure the following operator implementations in your YAML config:

```yaml
model:
  ops_implementation:
    attn_implementation: "sdpa"
    moe_implementation: "fused_npu"
    cross_entropy_loss_implementation: "npu"
    rms_norm_implementation: "npu"
    rotary_pos_emb_implementation: "npu"
    swiglu_mlp_implementation: "eager"
    load_balancing_loss_implementation: "eager"
    rms_norm_gated_implementation: "eager"
```

> **Note**: The above configuration is recommended for A5. Do **not** use `flash_attention_2` or `liger_kernel` backends on A5.

## Verification Status

| Category | A5 Status |
|---|---|
| Unit Tests (UT) | ✅ All passed |
| System Tests (ST) | ✅ All passed |
| CI Runner | A5 self-hosted runner (in preparation) |

For questions about A5 support, please open an [issue](https://github.com/ByteDance-Seed/VeOmni/issues) with the `ascend-npu` label.
