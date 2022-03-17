---
layout: post
title: "Clean Code 15장 - JUnit 들여다보기"
categories: Clean_Code
tags: [java, dev]
toc: true
---

- JUnit 프레임워크
- 결론

### JUnit 프레임워크
- JUnit에 대해서는 이미 충분히 알아보았다.
- 클린코드에서 몇번의 리팩토링을 거쳐 완성된 ComparisonCompactor.java를 한번 살펴보고 지나가자

```java
// 예제 코드 그대로 입력
import junit.framework.Assert;

public class ComparisonCompactor {
    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";

    private int contextLength;
    private String expected;
    private String actual;
    private int prefixLength;
    private int suffixLength;

    public ComparisonCompactor(int contextLength, String expected, String actual) {
        this.contextLength = contextLength;
        this.expected = expected;
        this.actual = actual;
    }

    public String formatCompactedComparison(String message) {
        String compactExpected = expected;
        String compactActual = actual;
        if (shouldBeCompacted()) {
            findCommonPrefixAndSuffix();
            compactExpected = compact(expected);
            compactActual = compact(actual);
        }
        return Assert.format(message, compactExpected, compactActual);
    }

    private boolean shouldBeCompacted() {
        return !shouldNotBeCompacted();
    }

    private boolean shouldNotBeCompacted() {
        return expected == null || actual == null || expected.equals(actual);
    }

    private void findCommonPrefixAndSuffix() {
        findCommonPrefix();
        suffixLength = 0;

        for (; !suffixOverlapsPrefix(); suffixLength++) {
            if (charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength))
                break;
        }
    }

    private char charFromEnd(String s, int i) {
        return s.charAt(s.length() - i - 1);
    }

    private boolean suffixOverlapsPrefix() {
        return actual.length() - suffixLength <= prefixLength ||
                expected.length() - suffixLength <= prefixLength;
    }

    private void findCommonPrefix() {
        prefixLength = 0;
        int end = Math.min(expected.length(), actual.length());
        for (; prefixLength < end; prefixLength++)
            if (expected.charAt(prefixLength) != actual.charAt(prefixLength))
                break;
    }

    private String compact(String s) {
        return new StringBuilder()
                .append(startingEllipsis())
                .append(startingContext())
                .append(DELTA_START)
                .append(delta(s))
                .append(DELTA_END)
                .append(endingContext())
                .append(endingEllipsis())
                .toString();
    }

    private String startingEllipsis() {
        return prefixLength > contextLength ? ELLIPSIS : "";
    }

    private String startingContext() {
        int contextStart = Math.max(0, prefixLength - contextLength);
        int contextEnd = prefixLength;
        return expected.substring(contextStart, contextEnd);
    }

    private String delta(String s) {
        int deltaStart = prefixLength;
        int deltaEnd = s.length() - suffixLength;
        return s.substring(deltaStart, deltaEnd);
    }

    private String endingContext() {
        int contextStart = expected.length() - suffixLength;
        int contextEnd = Math.min(contextStart + contextLength, expected.length());
        return expected.substring(contextStart, contextEnd);
    }

    private String endingEllipsis() {
        return (suffixLength > contextLength ? ELLIPSIS : "");
    }

}
```

- 클린코드에서 보이스카우트 규칙을 얘기하니 나도 좀 더 고쳐야겠다.

### 결론
- JUnit을 잘못 만들었다는게 아니다.
- 세상에 개선이 불필요한 모듈은 없다.
- 코드를 처음보다 조금 더 깨끗하게 만드는 책임은 우리 모두에게 있다.
