# Audit Report — `0xBBBBBbbBBb9cC5e90e3b3Af64bdAF62C37EEFFCb`

**Scanned:** 2026-08-21 · **Chain:** Ethereum mainnet

**Status:** FINDINGS


| Severity (after noise filter) | Count |
|---|---|
| Critical/High | 2 |
| Medium | 13 |
| Low/Informational (real) | 16 |
| Excluded as tool-noise (name-reused, naming-convention, off-topic CI/CD semgrep, etc.) | 27 |

## Findings (real, after noise filter)

### Reentrancy: State change after external call — High
*Source: aderyn*

```
Changing state after an external call can lead to re-entrancy attacks.Use the checks-effects-interactions pattern to avoid this issue.
```

### Integer Arithmetic Bugs — High
*Source: mythril*

```
The arithmetic operator can overflow.
It is possible to cause an integer overflow or underflow in the arithmetic operation.
```

### incorrect-equality — Medium
*Source: slither*

```
Morpho._accrueInterest(MarketParams,Id) (src/Morpho.sol#483-509) uses a dangerous strict equality:
	- elapsed == 0 (src/Morpho.sol#485)
```

### reentrancy-no-eth — Medium
*Source: slither*

```
Reentrancy in Morpho.borrow(MarketParams,uint256,uint256,address,address) (src/Morpho.sol#235-266):
	External calls:
	- _accrueInterest(marketParams,id) (src/Morpho.sol#249)
		- borrowRate = IIrm(marketParams.irm).borrowRate(marketParams,market[id]) (src/Morpho.sol#488)
	State variables written after the call(s):
	- market[id].totalBorrowShares += shares.toUint128() (src/Morpho.sol#255)
	Morpho.ma…
```

### reentrancy-no-eth — Medium
*Source: slither*

```
Reentrancy in Morpho.withdrawCollateral(MarketParams,uint256,address,address) (src/Morpho.sol#323-342):
	External calls:
	- _accrueInterest(marketParams,id) (src/Morpho.sol#333)
		- borrowRate = IIrm(marketParams.irm).borrowRate(marketParams,market[id]) (src/Morpho.sol#488)
	State variables written after the call(s):
	- position[id][onBehalf].collateral -= assets.toUint128() (src/Morpho.sol#335)
	…
```

### reentrancy-no-eth — Medium
*Source: slither*

```
Reentrancy in Morpho._accrueInterest(MarketParams,Id) (src/Morpho.sol#483-509):
	External calls:
	- borrowRate = IIrm(marketParams.irm).borrowRate(marketParams,market[id]) (src/Morpho.sol#488)
	State variables written after the call(s):
	- market[id].totalBorrowAssets += interest.toUint128() (src/Morpho.sol#490)
	Morpho.market (src/Morpho.sol#60) can be used in cross function reentrancies:
	- Morp…
```

### reentrancy-no-eth — Medium
*Source: slither*

```
Reentrancy in Morpho.setFee(MarketParams,uint256) (src/Morpho.sol#123-136):
	External calls:
	- _accrueInterest(marketParams,id) (src/Morpho.sol#130)
		- borrowRate = IIrm(marketParams.irm).borrowRate(marketParams,market[id]) (src/Morpho.sol#488)
	State variables written after the call(s):
	- market[id].fee = uint128(newFee) (src/Morpho.sol#133)
	Morpho.market (src/Morpho.sol#60) can be used in cr…
```

### reentrancy-no-eth — Medium
*Source: slither*

```
Reentrancy in Morpho.liquidate(MarketParams,address,uint256,uint256,bytes) (src/Morpho.sol#347-417):
	External calls:
	- _accrueInterest(marketParams,id) (src/Morpho.sol#358)
		- borrowRate = IIrm(marketParams.irm).borrowRate(marketParams,market[id]) (src/Morpho.sol#488)
	State variables written after the call(s):
	- market[id].totalBorrowShares -= repaidShares.toUint128() (src/Morpho.sol#385)
	Mo…
```

### reentrancy-no-eth — Medium
*Source: slither*

```
Reentrancy in Morpho.withdraw(MarketParams,uint256,uint256,address,address) (src/Morpho.sol#200-230):
	External calls:
	- _accrueInterest(marketParams,id) (src/Morpho.sol#214)
		- borrowRate = IIrm(marketParams.irm).borrowRate(marketParams,market[id]) (src/Morpho.sol#488)
	State variables written after the call(s):
	- market[id].totalSupplyShares -= shares.toUint128() (src/Morpho.sol#220)
	Morpho.…
```

### reentrancy-no-eth — Medium
*Source: slither*

```
Reentrancy in Morpho.supply(MarketParams,uint256,uint256,address,bytes) (src/Morpho.sol#169-197):
	External calls:
	- _accrueInterest(marketParams,id) (src/Morpho.sol#181)
		- borrowRate = IIrm(marketParams.irm).borrowRate(marketParams,market[id]) (src/Morpho.sol#488)
	State variables written after the call(s):
	- market[id].totalSupplyShares += shares.toUint128() (src/Morpho.sol#187)
	Morpho.mark…
```

### reentrancy-no-eth — Medium
*Source: slither*

```
Reentrancy in Morpho.repay(MarketParams,uint256,uint256,address,bytes) (src/Morpho.sol#269-298):
	External calls:
	- _accrueInterest(marketParams,id) (src/Morpho.sol#281)
		- borrowRate = IIrm(marketParams.irm).borrowRate(marketParams,market[id]) (src/Morpho.sol#488)
	State variables written after the call(s):
	- market[id].totalBorrowShares -= shares.toUint128() (src/Morpho.sol#287)
	Morpho.marke…
```

### uninitialized-local — Medium
*Source: slither*

```
Morpho.liquidate(MarketParams,address,uint256,uint256,bytes).badDebtShares (src/Morpho.sol#390) is a local variable never initialized
```

### uninitialized-local — Medium
*Source: slither*

```
Morpho.liquidate(MarketParams,address,uint256,uint256,bytes).badDebtAssets (src/Morpho.sol#391) is a local variable never initialized
```

### uninitialized-local — Medium
*Source: slither*

```
Morpho._accrueInterest(MarketParams,Id).feeShares (src/Morpho.sol#493) is a local variable never initialized
```

### unused-return — Medium
*Source: slither*

```
Morpho.createMarket(MarketParams) (src/Morpho.sol#150-164) ignores return value by IIrm(marketParams.irm).borrowRate(marketParams,market[id]) (src/Morpho.sol#163)
```

## Excluded (tool-noise, not vulnerabilities)

Listed for transparency — these were flagged by the underlying tools but are style/CI-artifact detectors, not security findings:

- `Centralization Risk`
- `Unspecific Solidity Pragma`
- `assembly`
- `low-level-calls`
- `naming-convention`
- `pragma`
- `solc-version`
