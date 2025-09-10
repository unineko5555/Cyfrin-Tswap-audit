# T-Swap Audit Project - 学習メモ

## Stateful Fuzz Testing & Handler実装の学び (2025-01-27)

### 🎯 Handler.t.sol実装での重要な発見

#### 1. foundry.toml設定の重要性

```toml
[fuzz]
seed = "0x1"  # 32バイトの16進文字列が必要、単純な0x2だとエラー
```

**学習ポイント**: Foundryの設定ファイルは厳密な型チェックあり

#### 2. Ghost Variables設計の複雑さ

- **actualDeltaX/Y**: 実際のプール残高変化
- **expectedDeltaX/Y**: AMM数式による期待値
- **startingX/Y**: 操作前のベースライン

**重要**: `address(this)` vs `address(pool)` の追跡対象間違いは致命的

#### 3. bound()関数の適切な使用

```solidity
// ❌ 間違い: 0を含むとTSwapPool__MustBeMoreThanZeroエラー
wethAmount = bound(wethAmount, 0, type(uint64).max);

// ✅ 正しい: プロトコル制約に従う
wethAmount = bound(wethAmount, pool.getMinimumWethDepositAmount(), type(uint64).max);
```

#### 4. デバッグプロセスのベストプラクティス

1. **-vvvv フラグ**: 詳細トレースで根本原因特定
2. **段階的修正**: 一度に1つの問題を解決
3. **エラー歓迎**: 新しいエラー = 前の問題解決の証拠
4. **反復アプローチ**: 完璧を最初から求めない

### 🚨 Handler実装でよくある間違い

#### 問題1: 不適切なgetInputAmountBasedOnOutput呼び出し

```solidity
// ❌ 致命的バグ
pool.getInputAmountBasedOnOutput(
    outputWeth,
    poolToken.balanceOf(address(pool)),
    type(uint64).max  // ❌ 実際のreservesではない！
);

// ✅ 正しい実装
pool.getInputAmountBasedOnOutput(
    outputWeth,
    poolToken.balanceOf(address(pool)),
    weth.balanceOf(address(pool))  // 正しいreserves
);
```

#### 問題2: 座標軸の不整合

同一Handlerクラス内で:

- deposit: X=poolToken, Y=WETH
- swap: X=WETH, Y=poolToken

**解決**: 一貫した座標系定義が必要

#### 問題3: アドレス分離の過度な複雑化

`liquidityProvider` vs `swapper` の分離は:

- **メリット**: 役割明確化、現実世界模倣
- **デメリット**: 管理複雑化、実質的価値は限定的
- **結論**: Handler内では単一アドレスで十分

### 🔧 実用的なテクニック

#### foundry.toml デバッグ設定

```toml
[fuzz]
seed = "0x1"  # 再現可能なテスト

[invariant]
runs = 256    # 十分な実行回数
depth = 32    # 操作チェーンの深さ
fail_on_revert = true
```

#### Ghost Variables パターン

```solidity
// 操作前記録
function _updateStartingDeltas(int256 wethAmount, int256 poolTokenAmount) internal {
    startingY = int256(poolToken.balanceOf(address(pool)));
    startingX = int256(weth.balanceOf(address(pool)));
    expectedDeltaX = wethAmount;
    expectedDeltaY = poolTokenAmount;
}

// 操作後比較
function _updateEndingDeltas() internal {
    actualDeltaX = int256(weth.balanceOf(address(pool))) - startingX;
    actualDeltaY = int256(poolToken.balanceOf(address(pool))) - startingY;
}
```

### 🎓 重要な学習成果

1. **Stateful Fuzz Testingは複雑**: 単純なunit testとは全く異なる思考が必要
2. **プロトコル制約の理解**: `MINIMUM_WETH_LIQUIDITY`等の細かい制約把握が重要
3. **デバッグスキル**: エラーメッセージから根本原因を追跡する能力
4. **反復改善**: 完璧を最初から求めず、段階的品質向上が現実的

## 次のステップ

実際の不変条件テスト(invariant_*)関数の実装へ

---

*更新: 2025-01-27 - Handler.t.solデバッグプロセス完了*