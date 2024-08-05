---
title: "Programmers 다리를 지나는 트럭"
author: hcbak
date: 2024-08-05 23:01:10 +0900
categories: [Algorithm, Programmers]
---

## 문제 이해
정해진 하중을 버틸 수 있는 다리가 있고, 주어진 여러 트럭들이 순서대로 다리를 지나고자 한다.

각각의 트럭에는 정해진 무게가 있고, 다리가 버틸 수 있는 하중 안에서 트럭들이 다리 위에 오를 수 있다.

정수 길이의 다리가 주어지면 트럭은 1초에 1씩 이동할 수 있고, 모든 트럭이 다리를 건널 때 까지 걸리는 시간을 리턴하면 된다.

## 입출력 조건 확인
아직까지 조건 계산(시간 & 공간 복잡도)이 어려운 것 같다.

일단 반복문 하나로 처리하기 때문에 문제가 없을 것이라고 생각한다.

## 문제 풀이 내용 정리
다리의 크기만큼 큐에다가 빈 칸(-1)을 넣어줬다.

반복문이 실행될 때 마다 시간(time)을 1초씩 증가시킨다.

트럭이 다리에 오르면 큐에 트럭의 무게를 넣고 다리가 견디고 있는 하중(bridgeWeight)이 증가한다.

만약 다리가 견디고 있는 하중(bridgeWeight)과 진입하는 트럭의 무게를 더했을 때 다리가 버틸 수 있는 하중(weight)을 초과하는 경우 트럭은 진입하지 않고 빈 칸(-1)이 진입한다.

트럭이 다리를 건너면 다리가 견디고 있는 하중을 다리를 건넌 트럭의 무게만큼 빼준다.

마지막 트럭이 다리에 오르면 그 트럭이 나올 때 까지 1초에 한 번 poll을 진행하고 마지막 트럭이 다리를 건너면 반복문이 종료된다.

### Java
```java
import java.util.Queue;
import java.util.LinkedList;

class Solution {
    public static int solution(int bridge_length, int weight, int[] truck_weights) {
        Queue<Integer> bridgeQ = new LinkedList<>();
        int[] result = new int[truck_weights.length];
        int bridgeWeight = 0;
        int time = 0;
        int index = 0;
        int tIndex = 0;

        for (int i = 0; i < bridge_length; i++)
            bridgeQ.offer(-1);

        while (true) {
            time++;

            if (bridgeQ.peek() == -1)
                bridgeQ.poll();
            else {
                bridgeWeight -= bridgeQ.peek();
                result[tIndex] = bridgeQ.poll();
                tIndex++;
            }

            if (index == truck_weights.length) {}
            else if (bridgeWeight + truck_weights[index] <= weight) {
                bridgeQ.offer(truck_weights[index]);
                bridgeWeight += truck_weights[index];
                index++;
            } else
                bridgeQ.offer(-1);
            
            if (tIndex == truck_weights.length)
                break;
        }

        return time;
    }
}
```

## 코드 문제점 및 해결 방법
원래 도저히 여기에 큐를 적용할 방법을 몰라서 이틀을 생각만 했다. 사실 원래 하던 대로 그냥 막 했으면 바로 풀었을 것 같은데 결과적으로 큐를 사용한 방법으로 풀게 되어 뿌듯했다!  
(원래 하려던 방법은 다리 길이 만큼 배열을 만들고 한 칸씩 이동시키는 직관적? 인 방법 ㅋㅋㅋㅋ 이젠 그만 해야해...)

Java의 특성을 살려 객체 지향으로 아름답게 푼 사람이 있는데 자세히는 안봤지만 다음에 문제를 풀 때 객체 지향적인 방향으로도 생각해볼 필요가 있을 것 같다.
