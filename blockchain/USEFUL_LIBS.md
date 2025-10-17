# Thư viện hữu ích cho Smart Contracts (Payroll / HR)

Dưới đây là danh sách các thư viện và công cụ hữu ích, được dùng nhiều khi xây hệ thống quản lý nhân sự trên blockchain (whitelist, phân quyền, claim lương, v.v.). Kèm theo lý do nên dùng và một vài lưu ý ngắn gọn.

---

## Mục lục ngắn (Quick Index)

Dạng ngắn gọn: Tên — mô tả 1 dòng — ví dụ ứng dụng — [link]

- [**OpenZeppelin Contracts**](#1-openzeppelin-contracts) — Thư viện hợp đồng chuẩn, nhiều modules (ERCs, utils) — token, phân quyền, bảo mật — [link](https://github.com/OpenZeppelin/openzeppelin-contracts)
- **OpenZeppelin Upgrades** — Hỗ trợ upgradeable proxy pattern — cập nhật logic sau deploy — [link](https://github.com/OpenZeppelin/openzeppelin-upgrades)
- [**SafeERC20**](#2-safeerc20-token-salary) — Wrapper an toàn cho ERC20 transfers — trả lương bằng token — [link](https://docs.openzeppelin.com/contracts/4.x/api/token#SafeERC20)
- [**AccessControl**](#3-accesscontrol--multirole) — Phân quyền theo role (non-owner) — HR/PAYROLL/PAUSER roles — [link](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl)
- [**MerkleProof**](#4-merkleproof-whitelist) — Verify Merkle proof on-chain — whitelist nhân viên / bulk pay — [link](https://docs.openzeppelin.com/contracts/4.x/api/utils#MerkleProof)
- [**ECDSA / EIP-712**](#5-ecdsa--eip-712-signed-off-chain-approvals) — Signature verification & typed signing — off-chain approvals / signed payouts — [link](https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA) / [link](https://eips.ethereum.org/EIPS/eip-712)
- [**PaymentSplitter**](#6-paymentsplitter--pull-pattern) — Chia thanh toán giữa nhiều địa chỉ — chia bonus/revenue — [link](https://docs.openzeppelin.com/contracts/4.x/api/finance#PaymentSplitter)
- [**TimelockController / Gnosis Safe**](#7-timelockcontroller--gnosis-safe-governance--multisig) — Delay + multisig cho admin actions — bảo vệ config/withdraw — [link](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController) / [link](https://gnosis-safe.io/)
- [**EnumerableSet / Counters / SafeCast**](#8-utility-helpers) — Utility helpers — quản list, id, safe casting — [link](https://docs.openzeppelin.com/contracts/4.x/api/utils)
- [**Foundry (forge)**](#9-security--dev-tools) — Fast Rust-based Solidity toolchain — test, fuzz, scripting — [link](https://github.com/foundry-rs/foundry)
- [**Hardhat**](#9-security--dev-tools) — JS-based dev environment với nhiều plugin — scripting, verify, gas reports — [link](https://hardhat.org/)
- [**Slither**](#9-security--dev-tools) — Static analysis cho Solidity — CI security checks — [link](https://github.com/crytic/slither)
- [**Echidna**](#9-security--dev-tools) — Fuzzer cho smart contracts — invariant fuzzing — [link](https://github.com/crytic/echidna)
- [**Tenderly**](#9-security--dev-tools) — Simulation & monitoring platform — debug tx, monitoring — [link](https://tenderly.co/)

## 1. OpenZeppelin Contracts

- Modules cần nắm: `AccessControl` / `AccessControlEnumerable`, `Ownable`, `Pausable`, `ReentrancyGuard`, `SafeERC20`, `PaymentSplitter`, `TimelockController`, `MerkleProof`, `ECDSA`, `Counters`.
- Vì sao: battle‑tested, được audit rộng rãi, cung cấp pattern chuẩn cho role/permission, bảo mật token transfer, timelock/multisig, proof-based whitelist, signature verification.
- Cài: `npm i @openzeppelin/contracts` hoặc `forge install OpenZeppelin/openzeppelin-contracts`.

## 2. SafeERC20 (token salary)

- Dùng để transfer token một cách an toàn (hỗ trợ token không trả boolean).
- Pattern: lưu `claimable` (mapping) → user gọi `claim()` để rút (pull pattern) thay vì push.

## 3. AccessControl / Multirole

- Dùng để phân tách role: `HR_ROLE`, `PAYROLL_ROLE`, `PAUSER_ROLE`, `ADMIN_ROLE`.
- Ưu điểm: granular permission, dễ audit, tránh single-owner anti-pattern.

## 4. MerkleProof (whitelist)

- Lưu `merkleRoot` trên-chain; dùng proof để verify whitelist cho nhiều addresses với gas rẻ.
- Thích hợp cho bulk airdrop/privileged lists.

## 5. ECDSA / EIP-712 (signed off-chain approvals)

- Cho phép ký lệnh off-chain (ví dụ: manager ký approval pay amount cho employee).
- Giảm on-chain ops, tiết kiệm gas, vẫn có audit trail.

## 6. PaymentSplitter / Pull pattern

- `PaymentSplitter` hữu ích khi chia phần thưởng/bonus giữa nhiều địa chỉ.
- Nhưng với payroll, prefer pull (store balances → employee claim) để tránh reentrancy/DoS khi push.

## 7. TimelockController + Gnosis Safe (governance & multisig)

- Dùng để bảo vệ các thay đổi cấu hình nhạy cảm (thay đổi rate, revoke role, update merkle root).
- Multisig cho các hành động admin cần nhiều ký.

## 8. Utility helpers

- `EnumerableSet`, `Counters`, `SafeCast` — quản lý danh sách, ID, casting an toàn.

## 9. Security & Dev tools

- Static analysis & audit: `Slither`.
- Test frameworks: `Foundry` (nhanh), `Hardhat` (flexible).
- Fuzzing: `Echidna`.
- Debugging & simulation: `Tenderly`.
- Upgradeable patterns: `@openzeppelin/contracts-upgradeable` (nếu cần proxy/upgrade).

## Mẫu kiến trúc ngắn (pattern khuyến nghị)

- Xương: `AccessControl` + `SafeERC20` + `ReentrancyGuard` + `MerkleProof`.
- Pattern rút tiền: Pull pattern (employee gọi `claim()` để rút salary).
- Bảo vệ admin: multisig + timelock cho thay đổi nhạy cảm.

## Ví dụ code ngắn (tổng quan)

- Allocate: HR tăng `claimable[employee]`.
- Set Merkle root: HR cập nhật `merkleRoot` cho whitelist.
- Claim: employee gọi `claim(amount, proof)` → kiểm tra proof nếu root tồn tại → chuyển token bằng `SafeERC20`.

## Tài nguyên & lệnh cài nhanh

- OpenZeppelin: https://github.com/OpenZeppelin/openzeppelin-contracts
- Foundry: https://github.com/foundry-rs/foundry
- Hardhat: https://hardhat.org/
- Cài nhanh (example):

```powershell
npm init -y
npm i --save-dev hardhat @nomicfoundation/hardhat-toolbox
npm i @openzeppelin/contracts
# hoặc với Foundry
# curl -L https://foundry.paradigm.xyz | bash; foundryup
```

---

© 2025 hwHoai | [github.com/hwHoai](https://github.com/hwHoai) | License: MIT
