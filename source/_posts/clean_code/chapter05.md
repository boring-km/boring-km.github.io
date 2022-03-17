---
layout: post
title: "Clean Code 5장 - 형식 맞추기"
categories: Clean_Code
tags: [java, dev]
toc: true
---

- 형식을 맞추는 목적
- 적절한 행 길이를 유지하기
- 가로 형식 맞추기
- 팀 규칙
- Uncle Bob의 형식 규칙



### 형식을 맞추는 목적

- 코드 형식은 너무 중요하다.
- 오늘 구현한 코드의 가독성은 앞으로 바뀔 코드의 품질에 지대한 영향을 미친다.
- 오랜 시간이 지나 원래 코드의 흔적을 더 이상 찾아보기 어려울 정도로 코드가 바뀌어도 맨 처음 잡아놓은 구현 스타일과 가독성 수준은 유지보수 용이성과 확장성에 계속 영향을 미친다.



### 적절한 행 길이를 유지하라

- 500줄을 넘지 않고 대부분 200줄 정도인 파일로도 커다란 시스템을 구축할 수 있다는 사실이다.

#### *신문 기사처럼 작성하라*

- 독자는 위에서 아래로 기사를 읽는다.
- 이름은 간단하면서도 설명이 가능하게 짓는다.
- 이름만 보고도 올바른 모듈을 살펴보고 있는지 아닌지를 판단할 정도로 신경 써서 짓는다.
- 소스 파일 첫 부분은 고차원 개념과 알고리즘을 설명한다.
- 아래로 내려갈수록 의도를 세세하게 묘사한다.
- 마지막에는 가장 저차원 함수와 세부 내역이 나온다.
- **한 면을 다 채우는 신문 기사는 거의 없다**
- **신문은 다양한 기사로 이뤄진다**
- **대다수 기사가 아주 짧다**

#### *개념은 빈 행으로 분리하라*

- 거의 모든 코드는 왼쪽에서 오른쪽으로 그리고 위에서 아래로 읽힌다.
- 각 행은 수식이나 절을 나타내고, 일련의 행 묶음은 완결된 생각 하나를 표현한다.
- **줄바꿈**: 생각 사이는 빈 행을 넣어 분리해야 마땅하다.

#### *세로 밀집도*

- 서로 밀접한 코드 행은 세로로 가까이 놓여야 한다.
- 세로 밀집도 예시

```java
public class ReporterConfig {
    /**
     * 리포터 리스너의 클래스 이름
     */
    private String m_className;

    /**
     * 리포터 리스너의 속성
     */
    private List<Property> m_properties = new ArrayList<Property>();
    public void addProperty(Property property) {
        m_properties.add(property);
    }
}
```

```java
public class ReporterConfig {
    private String m_className;
    private List<Property> m_properties = new ArrayList<Property>();
    
    public void addProperty(Property property) {
        m_properties.add(property);
    }
}
```

- 시선이 분산된다.

#### *수직 거리*
- 함수 연관 관계와 동작 방식을 이해하려고 이 함수에서 저 함수로 오가며 소스 파일을 위아래로 뒤지는 혼란을 어떻게 할까?
- 서로 밀접한 개념은 세로로 가까이 둬야 한다.
- 물론 두 개념이 서로 다른 파일에 속한다면 규칙이 통하지 않는다.
- 하지만 타당한 근거가 없다면 서로 밀접한 개념은 한 파일에 속해야 마땅하다.
- 한 개념을 이해하는 데 다른 개념이 중요한 정도다.
- 연관성이 깊은 두 개념이 멀리 떨어져 있으면 코드를 읽는 사람이 소스 파일과 클래스를 여기저기 뒤지게 된다.
- **변수 선언**: 변수는 사용하는 위치에 최대한 가까이 선언한다.
- **지역 변수**는 각 함수 맨 처음에 선언한다.
- **인스턴스 변수**는 클래스 맨 처음에 선언한다.
  - 변수 간에 세로로 거리를 두지 않는다.
  - **잘 설계한 클래스는 많은 (혹은 대다수) 클래스 메서드가 인스턴스 변수를 사용하기 때문이다. **
  - C++에서는 모든 인스턴스 변수를 클래스 마지막에 선언한다는 소위 **가위 규칙**을 적용한다.
  - 자바에서는 보통 클래스 맨 처음에 인스턴스 변수를 선언한다.
  - 중요한 건 변수 선언을 어디서 찾을지 모두가 알고 있어야 한다.
- **종속 함수**는  한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다.
  - 가능하다면 호출하는 함수를 호출되는 함수보다 먼저 배치한다.
  - 방금 호출한 함수가 잠시 후에 정의되리라는 사실이 예측된다.
- **개념적 유사성**: 어떤 코드는 서로 끌어당긴다.
  - ex) 한 함수가 다른 함수를 호출, 변수와 그 변수를 사용하는 함수, 비슷한 동작을 수행하는 일군의 함수
  - 명명법이 똑같고 기본 기능이 유사하고 간단하다.
  - 서로가 서로를 호출하는 관계는 부차적인 요인이다.

#### *세로 순서*

- **호출되는 함수를 호출하는 함수보다 나중에 배치한다.**
- 신문 기사와 마찬가지로 가장 중요한 개념을 가장 먼저 표현한다.



### 가로 형식 맞추기

- 짧은 행이 바람직하다.
- 홀러리스가 내놓은 80자 제한보다 좀더 넉넉하게 100자나 120자에 달해도 나쁘지 않지만 그 이상은 주의부족이다.

#### *가로 공백과 밀집도*

- 가로로는 공백을 사용해 밀접한 개념과 느슨한 개념을 표현한다.
- 예시

```java
private void measureLine(String line) {
  lineCount++;
  int lineSize = line.length();
  totalChars += lineSize;
  lineWidthHistogram.addLine(lineSize, lineCount);
  recordWidestLine(lineSize);
}
```

- 공백을 넣으면 두 가지 주요 요소가 확실히 나뉜다는 사실이 더 분명해진다.
- 반면, 함수 이름과 이어지는 괄호 사이에는 공백을 넣지 않았다.
- 함수와 인수는 서로 밀접하기 때문이다.
- 공백을 넣으면 한 개념이 아니라 별개로 보인다.
- 함수를 호출하는 코드에서 괄호 안 인수는 공백으로 분리한다.
- 쉼표를 강조해 인수가 별개라는 사실을 보여준다.

#### *가로 정렬*

- 공백을 일정하게 주는 형식
- 코드가 엉뚱한 부분을 강조해 진짜 의도가 가려지기 때문에 좋지 않다.
- 변수 유형은 무시하고 변수 이름부터 읽게 되거나 할당 연산자보다 오른쪽 피연산자에 눈이 가게 된다.
- ![가로정렬](./img/chapter5_image.png)
- **사용하지 말자**

#### *들여쓰기*

- 범위(scope)로 이뤄진 계층을 표현하기 위해 우리는 코드를 들여쓴다.
- 들여쓰는 정도는 계층에서 코드가 자리잡은 수준에 비례한다.
- 클래스 정의처럼 파일 수준인 문장은 들여쓰지 않는다.
- 클래스 내 메서드는 클래스보다 한 수준 들여쓴다.
- 메서드 코드는 메서드 선언보다 한 수준 들여쓴다.
- 블록 코드는 블록을 포함하는 코드보다 한 수준 들여 쓴다.
- **구조가 한눈에 들어온다**
- **들여쓰기 무시하기** : 간단한 if문, 짧은 while 문, 짧은 함수에서 들여쓰기 규칙을 무시하고픈 유혹이 생긴다.
  - 원점으로 돌아가 들여쓰기를 넣자.

#### *가짜 범위*

- 때로는 빈 while 문이나 for 문을 접한다.
- 가능한 피하자



### 팀 규칙

- 팀에 속한다면 자신이 선호해야 할 규칙은 바로 팀 규칙이다.
- 팀은 한 가지 규칙에 합의해야 한다.
- 모든 팀원은 그 규칙을 따라야 한다.
- 그래야 소프트웨어가 **일관적인 스타일**을 보인다.
- 개개인이 맘대로 짜대는 코드는 당연히 피하자



### 밥 아저씨의 형식 규칙

- (책에 나와있는 내용이지만 일일이 직접 타이핑 해보며 형식을 몸으로 익혀보기)

```java
public class CodeAnalyzer implements JavaFileAnalysis {
  private int lineCount;
  private int maxLineWidth;
  private int widestLineNumber;
  private LineWidthHistogram lineWidthHistogram;
  private int totalChars;
  
  public CodeAnalyzer() {
    lineWidthHistogram = new LineWidthHistogram();
  }
  
  public static List<File> findJavaFiles(File parentDirectory) {
    List<File> files = new ArrayList<File>();
    findJavaFiles(parentDirectory, files);
    return files;
  }
  
  private static void findJavaFiles(File parentDirectory, List<File> files) {
    for (File file : parentDirectory.listFiles()) {
      if (file.getName().endsWith(".java"))
        files.add(file);
      else if (file.isDirectory())
        findJavaFiles(file, files);
    }
  }
  
  public void analyzeFile(File javaFile) throws Exception {
    BufferedReader br = new BufferedReader(new FileReader(javaFile));
    String line;
    while	((line = br.readLine()) != null)
      measureLine(line);
  }
  
  private void measureLine(String line) {
    lineCount++;
    int lineSize = line.length();
    totalChars += lineSize;
    lineWidthHistogram.addLine(lineSize, lineCount);
    recordWidestLine(lineSize);
  }
  
  public int getLineCount() {
    return lineCount;
  }
  
  public int getMaxLineWidth() {
    return maxLineWidth;
  }
  
  public int getWidestLineNumber() {
    return widestLineNumber;
  }
  
  public LineWidthHistogram getLineWidthHistogram() {
    return lineWidthHistoram;
  }
  
  public double getMeanLineWidth() {
    return (double)totalChars/lineCount;
  }
  
  public int getMedianLineWidth() {
    Integer[] sortedWidths = getSortedWidths();
    int cumulativeLineCount = 0;
    for (int width : sortedWidths) {
      cumulativeLineCount += lineCountForWidth(width);
      if (cumulativeLineCount > lineCount/2)
        return width;
    }
    throw new Error("Cannot get here");
  }
  
  private int lineCountForWidth(int width) {
    return lineWidthHistogram.getLinesforWidth(width).size();
  }
  
  private Integer[] getsortedWidths() {
    Set<Integer> widths = lineWidthHistogram.getWidths();
    Integer[] sortedWidths = (widths.toArray(new Integer[0]));
    Arrays.sort(sortedWidths);
    return sortedWidths;
  }
}
```

