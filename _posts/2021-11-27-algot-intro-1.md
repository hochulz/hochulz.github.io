---
title: "주식 백테스팅 툴 개발기록 (1)"
excerpt: "자체개발 툴의 필요성과 기초 설계 과정"
categories:
- 미정
tags:
- 국내주식
- 백테스팅
--- 
---

그 동안 국내주식 전 종목을 유니버스로 하는 day trading 전략을 백테스팅 하기 위해서 전략 하나하나마다 프로그램을 짜고 약간의 수정이나 파라미터 변경도 일일이 코드를 수정하고 다시 컴파일하는 매우 번거로운 과정을 거쳤다. 또한 프로그램 성능에 대한 고려 없이 당장의 백테스팅 결과만 얻기 위해 조잡하게 코딩한 결과 백테스팅 시간도 무척 오래 걸리는 문제가 있다.  

이미 상용화된 툴이 있기는 했지만 제공하는 백테스팅 기간이 짧았고 무엇보다 자유도가 매우 낮아서 원하는 전략을 마음껏 테스트 할 수 없는 단점이 있었다. 그리고 웹 기반이라 전략에 대한 보안성도 걱정해야 했다. 그래서 다양한 전략을 원하는 대로 효율적으로 테스트해볼 수 있는 프로그램을 직접 만들기로 하고 개발을 시작했는데 거의 3개월 동안 짬을 내서 삽질한 끝에 어느 정도 모양이 나왔다. 아직 손 봐야 할 곳이 많지만 그래도 큰 틀은 잡힌 듯 해서 소개해 본다.

일단 퍼포먼스가 중요했기 때문에 언어는 C++ 기반으로 하되 테스트할 전략마다 프로그램 자체를 수정하지 않고 YesLanguage나 TradeStation처럼 전략 스크립트를 작성해서 Script Engine이 이를 해석해서 실행하게 했다. Script 언어는 JavaScript를 사용한다.

또 하나의 중요한 포인트는 테스트할 과거 데이터들을 어떤 형식으로 관리하고 읽어 들이냐 인데 이 부분이 속도 면에서 굉장히 중요해서 고민을 많이 했다. 결론은 기존에 사용하던 DBMS가 데이터 자체를 저장하고 관리하는 데는 편하지만 overhead가 너무 커서 백테스팅 프로그램에서 직접 쓸 수는 없었다. 그래서 DB상의 데이터를 전부 serialization 해서 flat file 형태로 저장하고 데이터가 필요할 때마다 파일을 찾아서 읽게 했다. 속도 면에서 flat file을 읽는 것이 db에서 쿼리를 돌리는 것보다 몇 십 배 빨랐다.

추가적으로 백테스팅에 걸리는 시간을 줄여보려고 테스트 구간을 나눠서 멀티쓰레딩을 적용할까 생각해 봤는데 백테스팅처럼 순차적으로 데이터를 읽고 처리하는 작업에는 불가능함을 깨닫고 단순히 데이터를 읽는 부분과 처리하는 부분만 다른 쓰레드로 구현했다.

![Algot - ControllerWindow](https://lh3.googleusercontent.com/dkcpiurYlTX9Iq8BTfAwFTOZvkjzOE01GyiB1YcNUNqKPxQU1VcQ1gV6MEiepN0WyqFOFcEWNAzNRix-x8auMN7kb_EjxweGxxQ014q46n_E4psFIgHOSuxWd1c_C3jif5C8EzMuxBYHkNSni6l2Tpv1ueYHwouVF2pR9kB_4VAcdF2Fp6W8o3XToXH1jmx6wWb05cyyAHERIPbwDLdPuhxUzdyjYitsB6IkUIrh8ostAuRValTKrU0fRL9tS53sQ62hVsqLz0vIBTsHoI2atOus1_mhZduzfDcxjLU3fN0bmz-B_upsDQFWw9nl7T2mzxPCArrMrfiJEd0Q4QljJ0Suj-ZIjqI1ZGrURKc_ePlzU96RRS5VcQ508WbJnhrfpWWqx_qHDOLA0wWapSXs0lrRaMGwPPpUSP0g8tgecTwhKz_HVB50uAw3_C7mHFDexbDwCEDTIsBWWkgyErM72iV3q7g1kv4cD3fQj6fQYg-ft8v8bWaPewIQ6zqqRjTygleb0D5uIOKmbfCpKv4MsoVorG82_zhGL06rjSG5ms0AYA3E_uJyLMky3IRaGr0j25Cl6N520cSlXXCIvclTvz8sysHtYwqeZi1s7YKMLA_ecWNkDfayymK77g9CZ2xspH0qsJ2G5CKqji9A7xQXqJEZmMGLCxk_SRZng2hiUSjxqG7oNi04gKbNidoYd0skXVIj06qtALQHPDaV1suN_g=w1202-h932-no?authuser=0) 
[주 화면 모습] JavaScript로 전략을 작성하고 편집할 수 있다

![Algot - ResultDialog](https://lh3.googleusercontent.com/Zd_sySaymq_y8aOpALq17QjstfADutLTdKEJn_JnTpIalE9ov5x-AdnM575F-6aKW9nbObI7OdednJEibGWnax2CFWqyQbMDt3t4uQ9RM43OdzwHdu0pPLs_qpnqUq22q4MBvOHx63orh8NL5haWjI0NJAn-bYGY-o9MyVyZ9RWh-zoZPNweR7WUQ1rEX7xHaa40lc0SIqhDrqwEwr3YIjOBgt1SdKiAitU7OmscRAP5EQnjsy6E0aagv05JAMAuLujHP2ToRKfvwk8kQKhNseRVGIG5AgXgKCfXVh6Kf86vFw4ztf_1QDJGnEgRCZtVxOcWmUz5BRscXcKyembLowxRgU1tU4-NjB3hQe8RF07AAb0lRXqpfUtQIM5ky4jlLeaMUHxVWQDrMcpguKZJs24HdOYdCdPOYBxjUb0yqjbPS02G9gzqMiaeK9Slqgr9SCpD8tsPXt6VhE7j9pTi6Hxuld1g6G1176L4g3rnt_FHsOF7_6zZh6HLJHuANday_teJ6c-Vcek99A4kC-hbc8zfE4heNzlVeS7cj4ogP1gsIfo7Jita5Csfy5gMrqHR6vJ_MQZetSNO4tP5F-7H4fQvLPvYXANZmzmTPrYK11ZCYcS6p2Qk8lR-NaRVC3Ecs6WMqtEjPdPFn8mqUidBVNRkHGXTl_oK_lIcB30q-R2T8Vy3Y13QPvybWnU28DbF1IrTF7nbTwPIkZWFeVTecQ=w1202-h932-no?authuser=0)
[결과 화면 모습] 수익률 그래프와 매매 내역, 각종 성과 지표가 표시될 예정

프로그램의 중요한 기능들은 구현이 되었기 때문에 직접 백테스팅을 하면서 필요한 기능을 추가하고 수정할 예정이다. 다음에는 프로그램의 기능과 실제 전략의 테스트 결과도 올려보겠다.