# Chapter 03. 미리 보는 코딩 테스트 오답 노트

## 03-1 예상치 못한 음수 결과 해결하기

- int 형의 표현 범위를 초과하는 큰 숫자를 다룰 때는 long 형을 사용하는 것이 더 안전하다.

#### int 형과 long 형의 표현 범위 비교
```java
public class Main {
    public static void main(String[] args) {

        int a = 1000000000;
        a += 2000000000;
        System.out.println(a);

        long b = 1000000000;
        b += 2000000000;
        System.out.println(b);
    }
}
```

- 결과가 예상치 못한 음수로 출려된다면, 가장 먼저 자료형을 점검해봐라.

## 03-2 시간 초과의 원인을 찾아 해결하기

- Scanner와 System.out.print() 대신 BufferedReader와 BufferedWriter를 활용하면 처리 속도를 크게 향상할 수 있다.
- Scanner는 입력할 때마다 필요한 자료형으로 변환하는 과정을 거치므로 처리 속도가 느려질 수 있다.
- System.out.print()는 출력이 발생할 때마다 버퍼를 비우는 작업이 이뤄지므로 성능이 저하될 수 있다.


- BufferedReader는 입력을 버퍼에 저장한 후 데이터를 한 번에 읽어 오는 방식으로 I/O 작업 횟수를 줄여 성능을 향상한다.
- BufferedWriter는 출력할 데이터를 먼저 버퍼에 저장한 후 한 번에 출력하는 방식으로 성능을 개선한다.

## 03-3 인덱스에 의미 부여하여 풀어 보기

- 상황에 따라 인덱스에 해싱 개념을 적용하여 단순한 위치가 아니라 특정한 의미를 지닌 값으로 활용하면 문제를 더 쉽게 해결할 수 있다.

> A[1]의 의미
> 1. 몇 번째 데이터인지 순서를 의미하는 경우 -> 첫 번째 데이터를 저장한다.
> 2. 숫잣값으로 의미를 부여한 경우 -> 1이라는 값이 몇 개 있는지를 저장한다.

#### 인덱스에 의미를 부여한 대표 사례 - 계수 정렬
```java
public class Main {
    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));

        int N = Integer.parseInt(br.readLine());

        int[] count = new int[1001];

        StringTokenizer st = new StringTokenizer(br.readLine());

        for (int i = 0; i < N; i++) {
            int number = Integer.parseInt(st.nextToken());
            count[number]++;
        }
        
        br.close();

        for (int i = 0; i < 1000; i++) {
            if (count[i] != 0) {
                for (int j = 0; j < count[i]; j++) {
                    bw.write(i + " ");
                }
            }
        }

        bw.flush();
        bw.close();
    }
}
```

- 인덱스를 단순히 '몇 번째 순서'로만 생각하지 말고, 문제 상황에 따라 다양한 의미로 변환해 보는 연습이 중요하다.

## 03-4 나머지 연산의 중요성 알아보기

### 나머지 연산의 분배 법칙

- 나눗셈을 제외하고 덧셈, 뺄셈, 곱셈의 분배 법칙이 성립한다.

> 덧셈의 분배 법칙 성립 -> (A + B) % C = (A % C + B % C) % C
> - (20 + 6) % 3 = 2
> - (20 % 3 + 6 % 3) % 3 = 2

> 뺄셈의 분배 법칙 성립 -> (A - B) % C = (A % C - B % C) % C
> - (20 - 6) % 3 = 2
> - (20 % 3 - 6 % 3) % 3 = 2

> 곱셈의 분배 법칙 성립 -> (A * B) % C = (A % C * B % C) % C
> - (20 * 6) % 3 = 2
> - (20 % 3 * 6 % 3) % 3 = 2

> 나눗셈의 분배 법칙 성립 -> (A / B) % C != (A % C) / (B % C) % C
> - (20 / 6) % 3 = 0
> - (20 % 3) / (2 % 3) % 3 = 1

#### 50!을 10007로 나눈 나머지 구하기
```java
public class Main {
    public static void main(String[] args) throws IOException {

        long answer = 1;

        // 메모리 오버플로우 발생
        for (int i = 1; i < 51; i++) {
            answer = answer * i;
        }

        System.out.println(answer % 10007);
    }
}
```

#### 50!을 10007로 나눈 나머지 구하기 - 수정
```java
public class Main {
    public static void main(String[] args) throws IOException {

        long answer = 1;

        // 나머지 연산의 분배 법칙
        // 곱셈의 분배 법칙 성립 -> (49! * 50) % 10007 = (49! % 10007 * 50 % 10007) % 10007
        for (int i = 1; i < 51; i++) {
            answer = (answer * i) % 10007;
        }

        System.out.println(answer);
    }
}
```

- 일반적으로 문제에 '결괏값을 OO으로 나눈 나머지를 출력하세요.'라는 문구가 있을 경우, 정답을 구한 후 마지막에 나머지 연산을 한 번만 적용하는 방법은 위험할 수 있다.

## 03-5 정렬 기초 다지기

### 오름차순 정렬

#### Arrays.sort()로 오름차순 정렬하기
```java
public class Main {
    public static void main(String[] args) throws IOException {

        int[] arr = {5, 4, 3, 2, 1};

        Arrays.sort(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

### 내림차순 정렬

#### 내림차순 정렬 1
```java
public class Main {
    public static void main(String[] args) throws IOException {

        Integer[] arr = {1, 2, 3, 4, 5};

        Arrays.sort(arr, Collections.reverseOrder());

        System.out.println(Arrays.toString(arr));
    }
}
```

#### 내림차순 정렬 2
```java
public class Main {
    public static void main(String[] args) throws IOException {

        int[] arr = {5, 3, 2, 4, 1};

        negate(arr);
        Arrays.sort(arr);
        negate(arr);

        System.out.println(Arrays.toString(arr));
    }

    static void negate(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            arr[i] *= -1;
        }
    }
}
```

## 03-6 다중 조건 정렬 익히기

- 자바에서는 Comparable과 Comparator 인터페이스를 사용하여 다중 조건 정렬을 구현할 수 있다.

#### Comparable을 이용한 다중 조건 정렬
```java
public class Score implements Comparable<Score> {

    int english;
    int math;

    public Score(int english, int math) {
        this.english = english;
        this.math = math;
    }

    @Override
    public String toString() {
        return "Score{" +
                "english=" + english +
                ", math=" + math +
                '}';
    }

    @Override
    public int compareTo(Score o) {
        return 0;
    }
}

public class Main {
    public static void main(String[] args) throws IOException {

        List<Score> myArr = new ArrayList<>();

        myArr.add(new Score(80, 100));
        myArr.add(new Score(100, 50));
        myArr.add(new Score(70, 100));
        myArr.add(new Score(80, 90));

        Collections.sort(myArr);

        for (Score score : myArr) {
            System.out.println(score);
        }
    }
}

// 출력
// Score{english=100, math=50}
// Score{english=80, math=100}
// Score{english=80, math=90}
// Score{english=70, math=100}
```

#### Comparator를 이용한 다중 조건 정렬
```java
class ScoreComparator implements Comparator<Score> {

    @Override
    public int compare(Score o1, Score o2) {
        if (o1.math == o2.math)
            return o2.english - o1.english;

        return o2.math - o1.math;
    }
}

public class Main {
    public static void main(String[] args) throws IOException {

        List<Score> myArr = new ArrayList<>();

        myArr.add(new Score(80, 100));
        myArr.add(new Score(100, 50));
        myArr.add(new Score(70, 100));
        myArr.add(new Score(80, 90));

        Collections.sort(myArr, new ScoreComparator());

        for (Score score : myArr) {
            System.out.println(score);
        }
    }
}

// 출력
// Score{english=80, math=100}
// Score{english=70, math=100}
// Score{english=80, math=90}
// Score{english=100, math=50}
```

> Comparable과 Comparator의 사용 방식과 유연성에는 다음과 같은 차이점이 있다.
> - Comparable: 정렬 대상 클래스 내부. 한 클래스당 하나의 정렬 기준만 정의할 수 있음.
> - Comparator: 클래스 외부 별도 정의. 한 클래스에 여러 개의 정렬 기준을 정의할 수 있음.

## 03-7 이차원 ArrayList 사용하기

```java
class Edge {
    int endNode;
    int value;

    public Edge(int endNode, int value) {
        this.endNode = endNode;
        this.value = value;
    }
}

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer tokenizer = new StringTokenizer(reader.readLine());

        int N = Integer.parseInt(tokenizer.nextToken());
        int E = Integer.parseInt(tokenizer.nextToken());

        List<Edge> list[] = new ArrayList[N + 1];

        for (int i = 0; i < E; i++) {
            tokenizer = new StringTokenizer(reader.readLine());
            int s = Integer.parseInt(tokenizer.nextToken());
            int e = Integer.parseInt(tokenizer.nextToken());
            int v = Integer.parseInt(tokenizer.nextToken());

            list[s].add(new Edge(e, v));
        }

        for (int i = 0; i < list[1].size(); i++) {
            Edge tmp = list[1].get(i);
            int next = tmp.endNode;
            int value = tmp.value;
        }
    }
}
```
