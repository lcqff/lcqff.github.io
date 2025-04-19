---
layout: post
title: "DB에 대량 데이터 삽입하기"
author: pam
categories: [SOFO]
categories: [SOFO]
tags: [] 
image: https://github.com/user-attachments/assets/3532a0a1-9151-43c9-bbb0-01e362c061dd
description: "데이터베이스에 총 10만개의 데이터를 삽입하자."
featured: false
toc: false
summary: "성능 측정 놀이를 하기 위해 서버에 충분한 데이터가 필요하다. 총 10만 개의 데이터 삽입을 위해 CSV를 생성하고, LOAD DATA를 사용하여 데이터를 DB에 삽입한다."


---
# 개요

성능 측정 놀이를 하기 위해 서버에 충분한 데이터가 필요했다. 데이터 100만개를 넣어보려다가 나의 연약한 프리티어 CPU가 걱정이 되어 한 10만개 정도만 넣기로 했다.

넣을 데이터는 회원, 앨범, 트랙, 팔로우 데이터 각각 2만, 2만, 3만, 3만개이다.

대량의 데이터 삽입을 위해 데이터가 들어있는 CSV를 생성하고, LOAD DATA를 사용하여 CSV 안의 데이터를 DB에 삽입할 것이다.

<br>

# CSV 생성

총 10만개의 데이터 Row를 지닌 CSV를 생성하기 위해 파이썬을 사용했다. 

Faker 라이브러리를 사용하면 가짜 데이터를 랜덤하게 생성가능하다.

```python
import csv
import random
import uuid
from faker import Faker

fake = Faker()

# 고정 개수
NUM_ARTISTS = 20000
NUM_FOLLOWS = 30000
NUM_ALBUMS = 20000
NUM_TRACKS = 30000

def get_image_url(i):
    return f"https://picsum.photos/seed/image{i}/400/300"

# Artist
with open('artist.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerow(['id', 'uuid', 'name', 'email', 'link1', 'link2', 'description', 'provider', 'artist_image', 'role',
                     'is_deleted'])
    for i in range(1, NUM_ARTISTS + 1):
        name = fake.user_name()
        provider = random.choice(['google', 'kakao', 'naver']);
        writer.writerow([
            i + 1000,
            str(uuid.uuid4()),
            name,
            name + str(i) + "@" + provider + ".com",
            fake.url(),
            fake.url(),
            fake.text(max_nb_chars=100).replace('\n','  '),
            provider,
            get_image_url(i),
            'ROLE_USER',
            0
        ])

... 
```

기존 DB에 저장돼있는 데이터와 구분해주기 위해 id를 1000이상부터 저장하도록 했다. 이미지 파일 같은 경우 s3에 직접 저장하기 보다는 picsum를 사용하여 랜덤한 더미 이미지를 생성했다. (s3 저장용량도 돈이다)

![Image](https://github.com/user-attachments/assets/512da472-4b43-4861-bbc4-80c3d1ba2284)


위 코드를 실행하면 다음과 같은 csv 파일이 생성된다. 

<br>


# 데이터 이동


이제 생성된 csv 파일을 서버의 Docker 컨테이너 안으로 복사해주자.

### Ec2로 데이터 복사

> `scp -i {pem 파일 저장 경로}/sofo-dev.pem {csv 파일 저장 경로}/*.csv {EC2 User}@{EC2 IP}:{저장할 경로}`
> 

![Image](https://github.com/user-attachments/assets/3532a0a1-9151-43c9-bbb0-01e362c061dd)

Csv 파일들을 Ec2 내로 복사해왔다.

### Mysql Docker 내 데이터 복사

> `docker cp ./artist.csv soundflyer-db:/var/lib/mysql-files/`
> 

![Image](https://github.com/user-attachments/assets/1a8e0f8b-dddd-48af-b347-a2c0a8a51cdc)

이때 복사하는 경로는 `LOAD DATA` 명령어로 접근 가능한 폴더여야한다.

접근 가능한 폴더가 무엇인지는 위 명령어를 통해 확인할수 있다.

![Image](https://github.com/user-attachments/assets/48ca5177-509f-4186-be78-de26f4062851)

<br>

# 데이터 삽입

[MySQL LOAD DATA](https://getchan.github.io/data/mysql_load_data/)

`LOAD DATA`를 통하면 몇만개의 데이터를 굉장히 빠른 속도로 삽입할수 있다.

```sql
LOAD DATA INFILE '/var/lib/mysql-files/track.csv'
INTO TABLE track
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"' 
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(id, uuid, title, lyric, track_url, duration, album_id, is_deleted);
```
이때 사용하는 쿼리는 위와 같다. 


- csv의 컬럼 순서가 DB 테이블의 컬럼 정의 순서와 동일할 경우 마지막 줄의 (id, uuid, …)를 생략가능하지만 그렇지 않은 경우 꼭 붙여주도록 하자.
- 데이터 내에 기호를 주의하자.
    
    ![Image](https://github.com/user-attachments/assets/70253ea4-a08c-4217-aac8-18a92761a031)
    
    csv 파일이 위와 같이 인코딩 되어있기 때문에 각 튜플을 구분하기 위해 개행문자를 사용하고 있어 개행문자가 들어가면 새로운 튜플로 인식한다. 쿼리에서 사용하는 ,나 “또한 마찬가지이다.
    
- 첫번째 행은 제목행이기 때문에 이를 무시하기 위해 IGNORE 1 ROWS를 사용한다.
- 컬럼 타입 주의하자. 특히 DB에 Boolean이 TINYINT로 저장되어있는지 확인하자.
    
    > `ALTER TABLE artist MODIFY COLUMN is_deleted TINYINT(1) NOT NULL DEFAULT 0;`
    > 
    
    만일 그렇지 않은 경우 변경해줘도 큰 문제 없다.