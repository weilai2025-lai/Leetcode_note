# Climbing Stairs (爬樓梯)

## 題目描述

**目標：** 假設你正在爬樓梯，需要 `n` 階才能到達樓頂。每次你可以爬 **1 階** 或 **2 階**。請問你有多少種不同的方法可以爬到樓頂？

---

## 硬體工程師視角

這題是經典中的經典，對應到你的硬體背景，就是 **「時序邏輯電路 (Sequential Logic)」**。

我們要計算當下的狀態，必須依賴「前一個時刻」跟「前前一個時刻」的狀態。這就像是你在寫 Verilog 時，用 **Flip-Flops (暫存器)** 鎖存住 $t-1$ 和 $t-2$ 的值，然後加起來變成 $t$ 時刻的輸出。

> [!NOTE]
> 這其實就是數學上的 **費氏數列 (Fibonacci Sequence)**。

---

## 第一步：變數與具體例子 (Variables & Examples)

我們不需要記住從第 1 階到第 100 階每一層的方法數，我們只需要記住「最近的兩階」就好。這就像硬體設計中的 **Shift Register (移位暫存器)**。

### 1. `n` (目標階數)
- **用途：** 我們要爬到的總高度
- **例子：** `n = 3`

### 2. `prev1` (前一階的方法數)
- **用途：** 這是指到達「我就在終點前一步 $(n-1)$」的方法數。因為從這裡，我只要跨 1 步就到了
- **硬體對應：** `Reg_A` ( $t-1$ 的值 )
- **例子：** 假設現在目標是第 3 階，這變數存的是「爬到第 2 階的方法數」

### 3. `prev2` (前兩階的方法數)
- **用途：** 這是指到達「我在終點前兩步 $(n-2)$」的方法數。因為從這裡，我可以選擇一次跨 2 步直接到終點
- **硬體對應：** `Reg_B` ( $t-2$ 的值 )
- **例子：** 假設現在目標是第 3 階，這變數存的是「爬到第 1 階的方法數」

### 4. `current` (當前階數的方法數)
- **用途：** `prev1 + prev2`
- **邏輯：** 要到第 3 階，我只能從第 2 階走上來，或是從第 1 階跳上來。沒有別的路了

---

## 第二步：演算法邏輯引導 (Algorithm Walkthrough)

我們從最簡單的情況開始推導（Bottom-Up），就像你在畫 **Timing Waveform** 一樣，一個 cycle 一個 cycle 往後推。

### 執行過程模擬

#### Base Case (基本狀況)
- **1 階樓梯：** 只有 1 種爬法 (1步) → `Ways(1) = 1`
- **2 階樓梯：** 有 2 種爬法 (1步+1步, 或是直接 2步) → `Ways(2) = 2`

#### 推導 n = 3
想要踩上第 3 階，你只能從 **第 2 階** 走 1 步上來，或者從 **第 1 階** 跨 2 步上來。

所以：
```
Ways(3) = Ways(2) + Ways(1)
```

計算：`2 + 1 = 3` 種方法

#### 推導 n = 4
想要踩上第 4 階，來源只能是 **第 3 階** 或 **第 2 階**。

所以：
```
Ways(4) = Ways(3) + Ways(2)
```

計算：`3 + 2 = 5` 種方法

#### 觀察規律

這就是一個累加器迴圈：

```
Current = Previous_1 + Previous_2
```

算完後，把數值往後推（**Shift**）：
- 舊的 `Previous_1` 變成新的 `Previous_2`
- 剛算出來的 `Current` 變成新的 `Previous_1`

---

### 費氏數列數值表

| 階數 n | Ways(n) | 公式 |
|--------|---------|------|
| 1 | 1 | Base case |
| 2 | 2 | Base case |
| 3 | 3 | 2 + 1 |
| 4 | 5 | 3 + 2 |
| 5 | 8 | 5 + 3 |
| 6 | 13 | 8 + 5 |
| 7 | 21 | 13 + 8 |

---

## 第三步：Python 程式碼實作

這段程式碼用最節省記憶體的方式（只用變數，不用 Array）來實作。

```python
def climbStairs(n):
    # 1. 處理基本邊界狀況 (Corner Cases)
    # 如果只有 1 階或 2 階，答案直接就是 n 本身 (1種或2種)
    if n <= 2:
        return n
    
    # 2. 初始化暫存器 (Registers Initialization)
    # 我們從第 3 階開始算，所以先設定好第 1 階和第 2 階的值
    # prev2 代表 "前兩階" (一開始是第 1 階的方法數：1)
    # prev1 代表 "前一階" (一開始是第 2 階的方法數：2)
    prev2 = 1 
    prev1 = 2
    
    # 3. 開始迴圈 (Sequential Logic)
    # 我們要算第 3 階到第 n 階
    for i in range(3, n + 1):
        
        # 4. 計算當前階數的方法數 (Next State Logic)
        # Current = (n-1)的方法數 + (n-2)的方法數
        current = prev1 + prev2
        
        # 5. 更新暫存器 (Shift Registers)
        # 就像時鐘訊號 (Clock Edge) 來了一樣，把值往後傳
        # 舊的 prev1 變成新的 prev2 (雖然在 Python 裡這兩行順序很重要)
        prev2 = prev1
        prev1 = current
        
        # Python 高手寫法 (上面三行可以縮減成一行)：
        # prev2, prev1 = prev1, prev1 + prev2
        
    # 6. 最後 prev1 裡面存的就是第 n 階的方法數
    return prev1
```

---

## 第四步：復健重點分析 (Key Takeaways)

### 1. 動態規劃 (DP) 與 硬體狀態機

很多人覺得 DP 很難，其實它的核心就是 **「不用重複計算已經算過的東西」**。

| 設計方式 | 硬體類比 | 特性 |
|----------|----------|------|
| **每次從頭算** | Combinational Logic (組合邏輯) | 延遲很大 |
| **存上個結果** | Sequential Logic (時序邏輯) | 高效率，這就是 DP！ |

在硬體裡，如果每個 Cycle 都要重新從 Input 算一遍，那是 Combinational Logic (組合邏輯)，延遲很大。

如果我們把上一個 Cycle 的結果存起來（Register），下一個 Cycle 直接拿來用，這就是 Sequential Logic (時序邏輯)，也是 **DP 的精髓**。

### 2. 空間優化 ($O(1)$ Space)

- **普通的 DP 解法：** 開一個陣列 `dp[n]` 來存每一階的答案
  - 硬體等價：要用一整塊 **SRAM**
  
- **優化後的解法：** 只用 `prev1`, `prev2` 兩個變數（兩個 Registers）
  - 硬體等價：極致的 **Area Optimization (面積優化)**

### 3. 多重賦值 (Tuple Unpacking)

在 Python 裡，`a, b = b, a+b` 是完全合法的，而且它是「同時」更新（**Parallel Assignment**）。

這跟 Verilog 的 **Non-blocking assignment (`<=`)** 概念非常像！

```python
# C 語言風格 (Sequential)
a = b       # a 先被改變
b = a + b   # b 會用到新的 a (錯誤！)
```

```python
# Python 多重賦值 (Parallel)
a, b = b, a+b  # 同時更新，使用舊值
```

```verilog
// Verilog Non-blocking (Parallel)
always @(posedge clk) begin
    a <= b;      // 使用舊的 b
    b <= a + b;  // 使用舊的 a 和 b
end
```

---

## 硬體對映 (The Hardware Mapping)

想像你的電路板上有兩顆 **D-Flip Flop (DFF)**，分別叫做 `Reg_1` 和 `Reg_2`。

```
┌─────────────────────────────────────────────┐
│  Fibonacci Sequence Generator Circuit       │
│                                              │
│     ┌──────┐      ┌──────┐                 │
│     │Reg_2 │      │Reg_1 │                 │
│     │prev2 │      │prev1 │                 │
│     └──┬───┘      └──┬───┘                 │
│        │             │                      │
│        │    ┌────┐   │                      │
│        └───►│ +  │◄──┘                      │
│             └─┬──┘                          │
│               │                             │
│               ▼                             │
│           current (Wire)                    │
│               │                             │
│      ┌────────┴────────┐                    │
│      │                 │                    │
│      ▼                 ▼                    │
│   To Reg_2         To Reg_1                │
│   (Shift)          (Update)                │
│                                              │
│  Clock ──────────┬────────┬─────────        │
│                  │        │                 │
│                  ▼        ▼                 │
│               [DFF_2]  [DFF_1]              │
└─────────────────────────────────────────────┘
```

| Python 變數 | 硬體元件 | 說明 |
|-------------|----------|------|
| `prev1` | `Reg_1` | 存最新的過去 |
| `prev2` | `Reg_2` | 存次新的過去 |
| `current` | `Adder` 的輸出線路 (Wire) | 組合邏輯計算結果 |

---

## 更新順序的重要性 (Data Hazard)

### 1. Python 的執行順序 vs. Verilog 的時序

這三行程式碼其實是在模擬「一個 Clock Cycle 發生後的狀態更新」。但因為軟體是逐行執行的，順序寫錯就會導致資料遺失（**Data Hazard**）。

```python
# 1. 組合邏輯 (Combinational Logic)
# 計算下一態 (Next State)
current = prev2 + prev1 

# 2. 更新暫存器 (Sequential Logic / Register Update)
# 關鍵：必須先由後往前搬！不然資料會被蓋掉！
prev2 = prev1    # 把 Reg_1 的舊值移進 Reg_2
prev1 = current  # 把 adder 的新值存進 Reg_1
```

### 2. 為什麼順序很重要？ (錯誤示範)

> [!CAUTION]
> 假如我們把順序顛倒（這是在寫軟體時常犯的錯）：

```python
# 錯誤示範 ❌
current = prev2 + prev1
prev1 = current  # <--- 先把 prev1 更新成新的值 (例如 3)
prev2 = prev1    # <--- 完蛋！這裡讀到的 prev1 已經是 3 了，原本的 2 不見了！
```

這就像你在 Verilog 裡面誤用了 **blocking assignment (`=`)** 而不是 **non-blocking (`<=`)**，導致 **Race Condition**。

**正確的軟體寫法必須是「後浪推前浪」：**
1. 先把舊的 `prev1` 安全地存到 `prev2`
2. 再把新的 `current` 寫入 `prev1`

### 3. 數值模擬 (Trace with Values)

我們實際帶數字跑一次，看著資料怎麼「流動」。

#### 初始狀態 (準備算第 3 階)
```
prev2 (Reg_2) = 1
prev1 (Reg_1) = 2
```

#### 執行程式碼

**Step 1: 計算 current**
```python
current = prev2 + prev1
```
- 計算：`1 + 2`
- `current` 變成 `3` (這是第 3 階的答案)
- 此時暫存器還沒變：`prev2=1, prev1=2`

**Step 2: 更新 prev2**
```python
prev2 = prev1  # 移位第一步
```
- 把 `prev1` 的 `2` 複製給 `prev2`
- 現在 `prev2` 變成 `2` 了
- 舊的 `1` 被丟掉了，因為我們不再需要它

**Step 3: 更新 prev1**
```python
prev1 = current  # 移位第二步
```
- 把 `current` 的 `3` 複製給 `prev1`
- 現在 `prev1` 變成 `3` 了

#### 結果 (準備算第 4 階)
```
prev2 (Reg_2) = 2
prev1 (Reg_1) = 3
```

**窗戶成功往右滑了一格！** ✓

---

## 視覺化：暫存器更新流程

```
算第 3 階時：

Before:               Calculate:           After Shift:
┌──────┐             ┌──────┐             ┌──────┐
│ prev2│             │ prev2│             │ prev2│
│  = 1 │             │  = 1 │             │  = 2 │ ← 從 prev1 搬過來
└──────┘             └──────┘             └──────┘
                          │
┌──────┐                  │               ┌──────┐
│ prev1│                  ├──►[Adder]     │ prev1│
│  = 2 │                  │      │        │  = 3 │ ← 從 current 搬過來
└──────┘             └────┘      │        └──────┘
                                  ▼
                            current = 3
                        (第3階的答案)

──────────────────────────────────────────────────

算第 4 階時：

Before:               Calculate:           After Shift:
┌──────┐             ┌──────┐             ┌──────┐
│ prev2│             │ prev2│             │ prev2│
│  = 2 │             │  = 2 │             │  = 3 │
└──────┘             └──────┘             └──────┘
                          │
┌──────┐                  │               ┌──────┐
│ prev1│                  ├──►[Adder]     │ prev1│
│  = 3 │                  │      │        │  = 5 │
└──────┘             └────┘      │        └──────┘
                                  ▼
                            current = 5
                        (第4階的答案)
```

---

## 時間與空間複雜度

- **時間複雜度：** $O(n)$ - 需要計算從第 3 階到第 n 階
- **空間複雜度：** $O(1)$ - 只使用兩個變數，常數空間（面積優化！）

---

## Verilog 程式碼對照

如果用硬體描述語言來實作這個邏輯：

```verilog
module climb_stairs (
    input wire clk,
    input wire rst,
    input wire [7:0] n,
    output reg [31:0] ways
);

reg [31:0] prev1, prev2;
reg [7:0] counter;

always @(posedge clk or posedge rst) begin
    if (rst) begin
        prev1 <= 2;
        prev2 <= 1;
        counter <= 3;
        ways <= 0;
    end else if (counter <= n) begin
        // Non-blocking assignment (並行更新)
        prev2 <= prev1;
        prev1 <= prev1 + prev2;
        counter <= counter + 1;
        ways <= prev1 + prev2;
    end
end

endmodule
```

注意這裡使用 **Non-blocking (`<=`)** 確保所有暫存器在同一個時鐘週期並行更新！

---

## 擴展練習

這個技巧可以應用到所有需要「記住前面狀態」的問題：

| 題目 | 說明 | DP 狀態 |
|------|------|---------|
| **Fibonacci Number** | 經典費氏數列 | `F(n) = F(n-1) + F(n-2)` |
| **House Robber** | 不能搶相鄰的房子 | `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` |
| **Min Cost Climbing Stairs** | 每階有不同成本 | `dp[i] = min(dp[i-1], dp[i-2]) + cost[i]` |

全部都可以用「兩個暫存器」的方式優化到 $O(1)$ 空間！
