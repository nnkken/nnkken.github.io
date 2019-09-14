---
title: "快速排序中分割元素的的另一種寫法"
type: post
date: 2019-04-17T00:30:00+08:00
---

快速排序（Quicksort）由以下幾個部分組成：

1. 選定 `pivot`
2. 移動數組中的元素，使元素以 `pivot` 為界分割成兩部分，前面部分全部小於 `pivot`，後面部分全部大於 `pivot`
3. 遞歸排序前面部分與後面部分

可見第二步的分割是快速排序中的重要部分，基本上所有移動元素的邏輯都由這部分負責。

所以……怎麼寫？

# 難以理解的左右指針

一般看到的快速排序寫法，都會使用以下我稱為「左右指針」的做法：定義兩個指針，一個從左往右掃描，直到找到比 pivot 大的元素；另一個從右到左掃描，直到找到比 pivot 小的元素；然後交換兩個指針的元素，並重覆掃描步驟，直到兩個指針相遇。

```C
void quicksort(int* start, int* end) {
    if (end - start < 2) {
        return;
    }
    int pivot = *start;
    int* left = start + 1;
    int *right = end - 1;
    while (left < right) {
        while (*left < pivot) {
            ++left;
        }
        while (*right > pivot) {
            --right;
        }
        int tmp = *left;
        *left = *right;
        *right = tmp;
        ++left;
        --right;
    }
    quicksort(start, right);
    quicksort(left, end);
}
```

以上是我隨便寫的代碼，百分百保證有 bug。

根據經驗，它可能會無限循環，或者爆棧（stack overflow），或者越界（segmentation fault），就算成功執行完畢，最後排序結果大概也不對。

當然，造成這種狀況的主要原因，是我寫得不好，或者說懶得寫好。然而我實在很難搞清楚怎麼寫好左右指針：

 - 到底指針初始值應該設成甚麼？`right` 似乎不需要考慮，但 `left` 應該設在 `pivot` 的位置（0），還是從下一個位置開始？
 - 到底內部循環應該用 `<` 還是 `<=` 作為終止條件？
 - 循環結束後，我們需要知道分割的邊界，那到底是 `left` 的左邊還是 `right` 的右邊？
 - 考慮到上一條，甚麼時候應該 `++left` `--right`？要加個 `if` 判斷嗎？還是直接搬到循環開頭？
 - 程序中有兩個指針，於是需要考慮的各項條件組合數目直接翻倍

我反反覆覆寫過幾遍快速排序，每次寫左右指針都像在煉蠱：首先嘗試在腦中模疑各種情況；十五分鐘後決定放棄，隨便寫個版本，然後開始邊測試邊修改，把 `>` `>=` `<` `<=` 的各種組合全部試一遍、隨便調整代碼順序、加點 `if` 避開奇怪的 edge case；經過無數次越界、爆棧、無限循環後，終於得出一段看着不知道在做甚麼，但運行結果貌似沒問題的代碼，然後把它供奉起來。至於註釋是不可能寫的，因為連我自己都不知道自己在做甚麼。

# 一種容易理解的寫法

有些人擅長處理這些複雜的東西，很容易就能寫出正確的版本；然而你一臉疑惑地問他們為甚麼要這樣寫、為甚麼這樣寫是正確的，他們也只能一臉疑惑地回答：這還要解釋？一看就知道這是對的呀。

大部分人（例如我）不擅長這樣的事，他們當中有些覺得編程 / 算法本來就是這麼難的，而且靠努力很難補償這種先天不足，於是放棄了。

而我從軟件工程中學到一件事：代碼是給人看的，而不只是給機器運行的；評估代碼的好壞，不能只看效能，還要看它能不能被人理解。換句話說，好的代碼應該有 `可理解性` (understandability)。

既然左右指針難以理解，我們就換一種分割的寫法。

```C
void quicksort(int* start, int* end) {
    if (end - start < 2) {
        return;
    }
    int pivot = *start;
    int* splitBoundary = start + 1;
    for (int* right = start + 1; right < end; right++) {
        int value = *right;
        if (value <= pivot) {
            *right = *splitBoundary;
            *splitBoundary = value;
            splitBoundary++;
        }
    }
    quicksort(start, splitBoundary);
    quicksort(splitBoundary, end);
}
```

我相信大部分人就算還未能理解上面的代碼，也會覺得它比左右指針簡單：

 - 比較短
 - 沒有雙重 loop
 - 只有一個
 - 用的是 for loop，從 `start + 1` 開始歷遍整個數組
 - 有個叫 `splitBoundary` 的變量，一看就知道那裡就是分割的邊界

它的原理很簡單：

一開始我們有一個很小的分割： `[start, start + 1)` 是比 `pivot` 小或者相等的部分，`[start + 1, start + 1)` 是較大的部分（初始時是空的），整個 `已分割部分` 為 `[start, start + 1)`（只有 `start`）。

然後我們每次都把 `已分割部分` 延伸一個元素，如果新元素比 `pivot` 大，就甚麼都不用做，自動落入右邊部分；否則我們把分割邊界上比 `pivot` 大的元素與新元素對調，並延伸左邊部分的邊界，讓新元素落入左邊部分。

> For every complex problem there is an answer that is clear, simple, and wrong.

上面的寫法沒有考慮一個問題：如果全部元素都小於或等於 `pivot`，整個程序就會無窮遞歸。

仔細想想，快速排序是怎樣確保遞歸會終止的？

分割結束後，既然我們已經確定了 `小於或等於 pivot` 和 `大於 pivot` 兩部分的邊界，我們也就確定了排序結束後 `pivot` 應該出現在哪裡：就在這個邊界的左邊。

而我們的分割寫法裡，`pivot` 從來沒有動過，一直在 `start` 的位置，難怪會出問題。

解決方法也很簡單：把 `pivot` 調到中間，然後遞歸時少包含一個位置：

```C
void quicksort(int* start, int* end) {
    if (end - start < 2) {
        return;
    }
    int pivot = *start;
    int* splitBoundary = start + 1;
    for (int* right = start + 1; right < end; right++) {
        int value = *right;
        if (value <= pivot) {
            *right = *splitBoundary;
            *splitBoundary = value;
            splitBoundary++;
        }
    }
    *start = *(splitBoundary - 1);
    *(splitBoundary - 1) = pivot;
    quicksort(start, splitBoundary - 1);
    quicksort(splitBoundary, end);
}
```

這種寫法不是我發明的，我也是從網上看到的；然而原代碼雖然沒有註釋，變量也是隨便命名，我卻很容易就看懂了，從此終於能一次編譯（或者兩次吧）寫出正確的快速排序。

而且這種寫法和左右指針一樣，都是 `O(n)` 的，因此對整個快速排序的複雜度沒有影響。