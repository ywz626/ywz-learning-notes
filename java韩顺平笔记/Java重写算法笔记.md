# 基本算法

## 快排

每次随机选择一个数 比该数小的在一侧大的在另一侧

然后递归排序两侧

```java
import java.util.Scanner;

public class Main {
    static int[] a = new int[100010];
    public static void sort(int l, int r)
    {
        if(l >= r) return;
        int mid = a[l+r>>1];
        int i = l-1, j = r+1;
        while(i<j)
        {
            do i++;while(a[i]<mid);
            do j--;while(a[j]>mid);
            if(i<j)
            {
                int temp = a[i];
                a[i] = a[j];
                a[j] = temp;
            }
        }
        sort(l,j);
        sort(j+1,r);
    }
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        for(int i = 1; i <= n; i++) {
            a[i] = sc.nextInt();
        }
        sort(1, n);
        for (int i = 1; i <= n; i++) {
            System.out.print(a[i] + " ");
        }
    }
}
```

