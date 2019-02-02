## 字符串的排列

##### 题目描述

输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。

##### 输入描述:

```
输入一个字符串,长度不超过9(可能有字符重复),字符只包括大小写字母。
```

![img](assets/7578108_1499250116235_8F032F665EBB2978C26C4051D5B89E90.png)

```

import java.util.ArrayList;
import java.util.Collections;

public class Solution {
    public ArrayList<String> Permutation(String str) {
        ArrayList<String> result = new ArrayList<>();
        if(str == null || str.length() ==0){
            return result;
        }
        findPermutation(str.toCharArray(),0,result);
        //结果要进行排序
        Collections.sort(result);
        return result;
    }

    private void findPermutation(char[] arr, int i, ArrayList<String> result) {
        //递归结束条件
        if(i == arr.length-1){
            //转为字符串
            String s = new String(arr);
            if(!result.contains(s)){
                result.add(s);
            }
        }else{
            for(int j= i; j<arr.length; j++){
                swap(arr, i, j);
                findPermutation(arr, i+1, result);
                swap(arr, i, j);
            }
        }

    }
    
    private void swap(char[] arr, int index, int j) {
        char tmp= arr[index];
        arr[index]= arr[j];
        arr[j]= tmp;
    }

    public static void main(String[] args) {
        String str = "abc";
        ArrayList<String> permutation = new Solution().Permutation(str);
        for (String s : permutation) {
            System.out.println(s);
        }
    }
}
```

## 数组中出现次数超过一半的数字

###### 题目描述

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

###### code

```
package Test28;

import org.jruby.ast.IterNode;
import org.junit.Test;

import java.util.Arrays;

/*
题目描述
数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。
例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。
由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如
果不存在则输出0。
 */
public class Test28 {
    @Test
    public void test() {
        int[] arr = new int[]{1, 2, 3, 2, 2, 2, 5, 4, 2};
        int i = MoreThanHalfNum_Solution(arr);
        System.out.println(i);
    }

    //方式一：时间复杂度O(N)
    public int MoreThanHalfNum_Solution(int[] array) {
        //先找到，然后判定是不是
        int count = 1;
        int tmp = array[0];
        for (int i = 0; i < array.length; i++) {
            if (tmp == array[i]) {
                count++;
            } else if (count > 0) { //不相等，次数>0 ，表示有重复的,
                count--;
            } else {
                tmp = array[i];
                count = 1;
            }
        }
        count=0;
        for(int i=0; i<array.length; i++){
            if(tmp== array[i]){
                count++;
            }
        }
        return count >array.length/2?tmp:0; //最后进行判定，是否是正确的
    }

	//方式二：
    //利用题意的规则：进行判断，先进行排序，然后判断中间值的那个是否正确
    //时间复杂度N*log(N)
//    public int MoreThanHalfNum_Solution(int[] array) {
//        if(array.length <1){
//            return 0;
//        }
//        int count = 0;
//        Arrays.sort(array);
//        int num = array[array.length/2];
//        for(int i=0; i<array.length; i++){
//            if(num == array[i])
//                count++;
//        }
//        if(count<= (array.length/2)){
//            num=0;
//        }
//        return num;
//    }
}

```

## 最小的K个数

##### 题目描述

>  输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

```

import java.util.ArrayList;
import java.util.Arrays;


/*
题目描述
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。
 */
public class Solution {
    public ArrayList<Integer> GetLeastNumbers_Solution(int[] input, int k) {
        ArrayList<Integer> result = new ArrayList<>();
        if (input == null || input.length == 0) {
            return result;
        }
        if(input.length<k){
            return result;
        }

        Arrays.sort(input);
        //存在k大于数组长度的情况
        for (int i = 0; i < k; i++) {
            result.add(input[i]);
        }
        return result;
    }
}
```

## 整数中1出现的次数（从1到n整数中1出现的次数）

##### 题目描述

> 求出1~13的整数中1出现的次数,并算出100~1300的整数中1出现的次数？为此他特别数了一下1~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数（从1 到 n 中1出现的次数）。

```
/*
题目描述
求出1~13的整数中1出现的次数,并算出100~1300的整数中1出现的次数？为此他特别数了一下1~13中
包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,
并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数（从1 到 n 中1出现的次数）。
 */
public class Solution {
    public int NumberOf1Between1AndN_Solution(int n) {
        int count=0;
        //遍历
        for (int i = n; i > 0; --i) {
            //转换成string字符串
            String str = String.valueOf(i);
            for (char c : str.toCharArray()) {
                if(c == '1'){
                    ++count;
                }
            }
        }
        return count;
    }
}
```

