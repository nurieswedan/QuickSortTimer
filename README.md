# Quick Sort Timing (Best vs Worst Case)

Java program that measures Quick Sort execution time in nanoseconds using `System.nanoTime()`.

## Scenarios

| Scenario | Array | Pivot | Complexity |
|---|---|---|---|
| Best case | Random, n = 1,000,000 | Middle element | O(n log n) |
| Worst case | Already sorted, n = 20,000 | Always last element | O(n^2) |

The worst case uses a smaller size because O(n^2) with 1,000,000 elements takes several minutes and the recursion depth reaches n (the program runs it in a thread with a large stack to avoid `StackOverflowError`).

## How it works

1. Read the time with `System.nanoTime()` and store it.
2. Run the sort.
3. Read the time again.
4. Elapsed time = second reading - first reading (in nanoseconds).

## Run

```bash
javac QuickSortTimer.java
java QuickSortTimer
```

Run it 3 times and take the average, because the first run is slower due to JIT warm-up.

## Code

```java
import java.util.Random;
import java.util.Arrays;
public class QuickSortTimer {
    static int[] randomArray(int n) {
        Random r = new Random();
        int[] a = new int[n];
        for (int i = 0; i < n; i++) a[i] = r.nextInt();
        return a;
    }
    // Best case: pivot in the middle
    static void quickSortBest(int[] a, int low, int high) {
        if (low >= high) return;
        int mid = low + (high - low) / 2;
        swap(a, mid, high);
        int p = partition(a, low, high);
        quickSortBest(a, low, p - 1);
        quickSortBest(a, p + 1, high);
    }
    // Worst case: pivot always the last element
    static void quickSortWorst(int[] a, int low, int high) {
        if (low >= high) return;
        int p = partition(a, low, high);
        quickSortWorst(a, low, p - 1);
        quickSortWorst(a, p + 1, high);
    }
    static int partition(int[] a, int low, int high) {
        int pivot = a[high];
        int i = low - 1;
        for (int j = low; j < high; j++) {
            if (a[j] <= pivot) { i++; swap(a, i, j); }
        }
        swap(a, i + 1, high);
        return i + 1;
    }
    static void swap(int[] a, int i, int j) {
        int t = a[i]; a[i] = a[j]; a[j] = t;
    }
    public static void main(String[] args) throws InterruptedException {
        // Scenario 1: best case, n = 1,000,000
        int[] best = randomArray(1_000_000);
        long start = System.nanoTime();
        quickSortBest(best, 0, best.length - 1);
        long end = System.nanoTime();
        System.out.println("Best: " + (end - start) + " ns");
        // Scenario 2: worst case, sorted array, last-element pivot
        int[] worst = randomArray(20_000);
        Arrays.sort(worst);
        final long[] result = new long[1];
        Thread t = new Thread(null, () -> {
            long s = System.nanoTime();
            quickSortWorst(worst, 0, worst.length - 1);
            result[0] = System.nanoTime() - s;
        }, "worst", 512L * 1024 * 1024);
        t.start(); t.join();
        System.out.println("Worst: " + result[0] + " ns");
    }
}
```
