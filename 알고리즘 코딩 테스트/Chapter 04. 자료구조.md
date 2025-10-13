# Chapter 04. 자료구조

## 04-1 배열과 리스트

### 배열
- 배열은 메모리의 연속 공간에 값이 채워져 있는 형태의 자료구조이다.

> 배열의 특징
> 1. 인덱스를 사용하여 값에 바로 접근할 수 있다.
> 2. 새로운 값을 삽입하거나 특정 인덱스에 있는 값을 삭제하기 어렵다. 값을 삽입하거나 삭제하려면 해당 인덱스 주변에 있는 값을 이동시키는 과정이 필요하다.
> 3. 배열의 크기는 선언할 때 지정할 수 있으며, 한 번 선언하면 크기를 늘리거나 줄일 수 없다.
> 4. 구조가 간단하므로 코딩 테스트에서 많이 사용한다.

### 리스트
- 리스트는 값과 포인터를 묶은 노드라는 것을 포인터로 연결한 자료구조이다.

> 리스트의 특징
> 1. 인덱스가 없으므로 값에 접근하려면 Head 포인터부터 순서대로 접근해야 한다. 다시 말해 값에 접근하는 속도가 느리다.
> 2. 포인터로 연결되어 있으므로 데이터를 삽입하거나 삭제하는 연산 속도가 빠르다.
> 3. 선언할 때 크기를 별도로 지정하지 않아도 된다. 다시 말해 리스트의 크기는 정해져 있지 않으며, 크기가 변하기 쉬운 데이터를 다룰 때 적절하다.
> 4. 포인터를 저장할 공간이 필요하므로 배열보다 구조가 복잡하다.

### 문제 001 숫자의 합 구하기

#### 1단계 문제 분석하기
- N의 범위가 1부터 100까지이므로 값을 int형, long형과 같은 숫자형으로 담을 수 없다.
- 문자열 형태로 입력값을 받은 후 이를 문자 배열로 변환하고, 문자 배열값을 순서대로 읽으면서 숫자형으로 변환하여 더해야 한다.

#### 2단계 손으로 풀어 보기

#### 3단계 Pseudo code 작성하기

#### 4단계 코드 구현하기
```java
public class Main {
    public static void main(String[] args) throws IOException {
        Scanner scanner = new Scanner(System.in);

        int N = scanner.nextInt();

        String sNum = scanner.next();
        char[] cNum = sNum.toCharArray();

        int sum = 0;

        for (int i = 0; i < cNum.length; i++) {
            sum += cNum[i] = '0';
        }

        System.out.print(sum);
    }
}
```

> 자바에서의 형 변환
> - String sNum = "1234";
> - int i1 = Integer.parseInt(sNum);
> - int i2 = Integer.valueOf(sNum);
> - double d1 = Double.parseDouble(sNum);
> - double d2 = Double.valueOf(sNum);
> - float f1 = Float.parseFloat(sNum);
> - float f2 = Float.valueOf(sNum);
> - long l1 = Long.parseLong(sNum);
> - long l2 = Long.valueOf(sNum);
> - short s1 = Short.parseShort(sNum);
> - short s2 = Short.valueOf(sNum);

### 문제 002 평균 구하기

#### 1단계 문제 분석하기
- 최고 점수를 기준으로 전체 점수를 다시 계산해야 하므로 모든 점수를 입력받은 후에 최고점을 별도로 저장해야 한다.
- 일일이 변환 점수를 구할 필요 없이 한 번에 변환한 점수의 평균 점수를 구할 수 있다.
- (A / M * 100 + B / M * 100 + C / M * 100) / 3 = (A + B + C) * 100 / M / 3

#### 2단계 손으로 풀어 보기

#### Pseudo code 작성하기

#### 코드 구현하기
```java
public class Main {
    public static void main(String[] args) throws IOException {

        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        int N = Integer.parseInt(reader.readLine());
        int[] arr = new int[N];

        StringTokenizer tokenizer = new StringTokenizer(reader.readLine());

        for (int i = 0; i < arr.length; i++) {
            arr[i] = Integer.parseInt(tokenizer.nextToken());
        }

        long max = 0;
        long sum = 0;

        for (int i = 0; i < arr.length; i++) {
            if (arr[i] > max) {
                max = arr[i];
            }

            sum += arr[i];
        }

        System.out.println((sum * 100.0) / (max * N));
    }
}
```

## 04-2 구간 합

- 구간 합은 합 배열을 이용하여 시간 복잡도를 더 줄이기 휘해 사용하는 특수한 목적의 알고리즘이다.
- 합 배열은 기존 배열을 전처리한 배열이라 생각하면 된다.
- 합 배열을 미리 구해 놓으면 기존 배열의 일정 범위 합을 구하는 시간 복잡도가 O(N)에서 O(1)로 감소한다.

### 문제 003 구간 합 구하기

#### 1단계 문제 분석하기
- 문제에서 수의 개수와 합을 구해야 하는 횟수는 최대 100,000이다.

#### 2단계 손으로 풀어 보기

#### 3단계 Pseudo code 작성하기

#### 4단계 코드 구현하기
```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));

        StringTokenizer tokenizer = new StringTokenizer(reader.readLine());

        int N = Integer.parseInt(tokenizer.nextToken());
        int M = Integer.parseInt(tokenizer.nextToken());

        int[] arr = new int[N + 1];

        tokenizer = new StringTokenizer(reader.readLine());

        for (int i = 1; i <= N; i++) {
            arr[i] = arr[i - 1] + Integer.parseInt(tokenizer.nextToken());
        }

        for (int q = 0; q < M; q++) {
            tokenizer = new StringTokenizer(reader.readLine());

            int i = Integer.parseInt(tokenizer.nextToken());
            int j = Integer.parseInt(tokenizer.nextToken());

            System.out.println(arr[j] - arr[i - 1]);
        }
    }
}
```
