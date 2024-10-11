---
layout: post
title: "bad SQL grammar []"
author: pam
categories: Backend-Study
tags: JDBC 오류
---

### 에러

```java
    public Long save(Reservation reservation) {
        final String sql = "insert into reservation (name, date, timeId) values (?, ?, ?)";

        jdbcTemplate.update(connection -> {
                    final PreparedStatement preparedStatement = connection.prepareStatement(sql, new String[]{"id"});
                    preparedStatement.setString(1, reservation.name());
                    preparedStatement.setString(2, reservation.date());
                    preparedStatement.setString(3, String.valueOf(reservation.timeId()));
                    return preparedStatement;
                }, keyHolder);
        return (Long) keyHolder.getKey();
    }
```

JDBC 템플릿을 통해 예약 데이터를 데이터베이스에 저장하는 실습을 진행하고 있었다. 오류가 발생한 코드는 위와 같다.


error: "Internal Server Error"  
message: <red>"PreparedStatementCallback; bad SQL grammar []"</red>   
path: "/reservations"  
status: 500  
timestamp: "2023-12-04T08:26:13.762+00:00"  
{: .box-error}

오류는 위와 같이 발생했다. SQL 문법이 잘못됐을 경우에는 보통 `bad SQL grammar [select * from member where name = ?]`와 같이 어떤 SQL이 잘못됐는지 알려줬기 때문에 SQL을 인식하지 못했다고 판단했다.


### 해결 과정
인터넷에 검색하니 나와 비슷한 오류를 겪은 인프런 [질문글](https://www.inflearn.com/questions/291010/%EC%95%88%EB%85%95%ED%95%98%EC%84%B8%EC%9A%94-%EC%98%81%ED%95%9C%EB%8B%98-%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%97%90%EC%84%9C-%EC%98%A4%EB%A5%98%EA%B0%80-%EB%82%A9%EB%8B%88%EB%8B%A4)이 있었다.

위 글의 답변은 아래와 같다.  

> encrypted_password 컬럼을 인식하지 못하는 것으로 보여집니다.  
작성한 쿼리 또는 DB에 해당 테이블의 컬럼이 정상적으로 생성되었는지 확인 부탁드려요.

위 답변으로 인해 컬럼 인식 문제라는 것을 알 수 있었다.

위 답변을 보고 데이터베이스를 확인했다.
<img alt="데이터베이스" src="https://github.com/lcqff/lcqff.github.io/assets/71930280/1b6fd2af-96aa-46bc-9403-a965ee1ff0eb">

데이터베이스에는 `time_id`값이 올바르게 생성된 것을 보아 정상적으로 작동했음을 알 수 있었다.


### 해결
데이터베이스가 문제가 아니니 코드상 SQL문의 칼럼 문제이다.  

다시 확인하니 `"insert into reservation (name, date, timeId) values (?, ?, ?)"`에서 time_id 칼럼이 스네이크 케이스가 아닌 카멜 케이스 timeId로 작성되어 있음을 확인할 수 있었다.  

이를 수정하니 해당 오류는 사라졌다. 앞으로 잘 확인하자...