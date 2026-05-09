# Hisilicon Crypto Patches Sync Summary

## Patches synced from OLK-6.6 to OLK-5.10

**Branch**: OLK-5.10-sync (based on origin/OLK-5.10)
**Date**: 2026-05-08
**GitHub**: https://github.com/IAMHCHCH/kernel-openeuler-510-patches

---

## Patch Overview

| # | Original Commit (OLK-6.6) | Patch Title | Module | Follow-up Patches Merged | OLK-5.10 Extra Modifications |
|---|---------------------------|-------------|--------|--------------------------|------------------------------|
| 1 | 522ef076b4a2 | zip - adjust the way to obtain the req in the callback function | zip | — | Kept `hisi_zip_ops_v1` for pre-V3 hardware; changed `hisi_zip_create_req(acomp_req, qp_ctx, head_size, is_comp)` → `hisi_zip_create_req(qp_ctx, acomp_req)` |
| 2 | 64de0460c306 | sec - move backlog management to qp and store sqe in qp for callback | sec2/qm | — | Added `#define SEC_RETRY_MAX_CNT 5U`; replaced OLK-5.10 `sec_bd_send` with OLK-6.6 retry loop approach; added `sec_alg_send_message_maybacklog` with `ret = sec_alg_try_enqueue(req)` |
| 3 | 99a85502372c | hpre - extend tag field to 64 bits for better performance | hpre | — | Restored `hpre_alloc_req_id` function body after conflict |
| 4 | df67841ddb91 | qm - centralize sending locks of each module into qm | qm/zip | — | Kept v1 ops struct (sqe_type=0) alongside renamed v2 ops |
| 5 | 0ec0adcc19f8 | qm - consolidate qp creation and start in hisi_qm_alloc_qps_node | qm/all | — | Added `qm_get_and_start_qp`; modified `hisi_qm_alloc_qps_node` to use centralized QP start |
| 6 | d43253c679bf | qm - add reference counting to queues for tfm kernel reuse | qm | — | Added `atomic_t refcnt` to qp; modified alloc/free paths |
| 7 | e9fbc14f3dcb | qm - optimize device selection priority based on queue ref count and NUMA distance | qm | — | Added NUMA-aware device selection logic |
| 8 | 6f1b51b4b265 | zip - support fallback for zip | zip | ✅ 085219c1c743 (fix callback) <br> ✅ bc7b8bfffff5 (add algorithm check) <br> ✅ eb446c04380e (remove redundant callback) | Added `hisi_zip_fallback_do_work`, `hisi_zip_fallback_init/uninit`, `fallback` flag, `soft_tfm`; `crypto_has_comp` check before alloc; error handling in callback |
| 9 | c0b36b175417 | hpre - support hpre algorithm fallback | hpre | — | Added `fallback` flag, `soft_tfm` for DH/RSA/ECDH; ECDH init simplified to call `hpre_ecdh_init_tfm` |
| 10 | c0f3bb7664a6 | sec2 - support skcipher/aead fallback | sec2 | — | Added `need_fallback` parameter to sec alg send paths |
| 11 | 78c79d98d54e | qm - lower priority for hisilicon crypto implementations | all | — | `cra_priority` reduced for hpre/sec2/zip algos |
| 12 | e8e54fd34780 | qm - mask all error type when uninit | qm/all | — | `QM_RAS_MASK_ALL` applied in *_uninit paths |

---

## Files Modified

| File | Patches Touching |
|------|------------------|
| drivers/crypto/hisilicon/qm.c | #2, #4, #5, #6, #7, #12 |
| include/linux/hisi_acc_qm.h | #2, #4, #5, #6 |
| drivers/crypto/hisilicon/zip/zip_crypto.c | #1, #4, #5, #8, #11 |
| drivers/crypto/hisilicon/sec2/sec_crypto.c | #2, #10, #11 |
| drivers/crypto/hisilicon/sec2/sec.h | #2 |
| drivers/crypto/hisilicon/hpre/hpre_crypto.c | #3, #9, #11 |
| drivers/crypto/hisilicon/hpre/hpre.h | #3 |
| drivers/crypto/hisilicon/Kconfig | #8 |
| drivers/crypto/hisilicon/hpre/hpre_main.c | #12 |
| drivers/crypto/hisilicon/sec2/sec_main.c | #12 |
| drivers/crypto/hisilicon/zip/zip_main.c | #12 |

---

## Key OLK-5.10 Compatibility Differences

1. **v1/v2 SQE ops**: OLK-5.10 maintains `hisi_zip_ops_v1` (sqe_type=0) for pre-V3 hardware alongside renamed v2 ops (`hisi_zip_ops`); OLK-6.6 consolidated to single ops struct (v2-only)
2. **Tag storage**: 32-bit (lower 16-bit sqe index) → full 64-bit pointer (`#define GET_REQ_FROM_SQE`)
3. **Backlog management**: Moved from per-module structures to centralized `qp->backlog` and `qp->msg[]` array in qm layer
4. **Function signatures**: `hisi_zip_create_req(acomp_req, qp_ctx, head_size, is_comp)` → `hisi_zip_create_req(qp_ctx, acomp_req)` (3 fewer params on OLK-6.6)
5. **sec_bd_send**: OLK-5.10 had different internal structure; replaced with OLK-6.6 retry-loop approach

---

## How to Apply

```bash
cd <kernel-tree>
git checkout OLK-5.10
git am /path/to/patches/0001-*.patch
git am /path/to/patches/0002-*.patch
# ... through 0012
```
