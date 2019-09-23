---
layout: post
title:  "C#에서 로컬 NoSql DB 사용하기"
subtitle:   "LiteDb 사용하기"
categories: develop
tags: csharp
---

## LiteDB란
각종 프로그램을 만들다 보면 데이터를 저장 할 필요가 있다.

적은 양의 데이터면 text 파일에 저장해서 이용이 가능하지만  
데이터의 양이 많아지면 어림도 없는 방법이다.

보통 이런상황에서 사용하는 로컬db가 하나 있다.  
바로 SQLite 다.  

하지만 SQLite 는 기본적으로 sql 문으로 작동하는 rdb 이므로 c# 코드와의 결합이 힘든 부분이 있다.

SQLite 용 EF 등을 이용하면 해결이 가능하지만 추가로 설치할 필요가 있으니 불편 할 수 있다.

이런 상황에 이용 가능한 .NET 용 내장 NoSQL 데이터베이스가 있다.  
바로 [litedb](https://www.litedb.org/)다.

이름에서 볼 수 있듯이 매우 가벼운 db로 단일 dll 파일(350kb)만 있으면 사용 가능한 db 이다.

해당 DB 는 Nuget 에서 받아서 바로 사용 가능하다.

![버튼](/assets/img/dev/csharp/LiteDB/nuget.PNG)  

한번 사용법을 알아보자!

## 사용법
일단 DB에 저장할 데이터의 구조를 class 로 만든다.  
해당 class 가 RDB의 Table 과 같은 역할을 한다.
```csharp
public class User
{
    [BsonId] // RDB 의 PK 라고 생각하면 된다.
    public string Id { get; set; }

    public string Password { get; set; }

    public string Name { get; set; }
}
```

간단하게 user 정보를 저장 가능한 class 를 만들었다.

또한 설명을 위한 폼을 만들어 보았다.

![폼 이미지](/assets/img/dev/csharp/LiteDB/Form1.png)  

폼 설명을 하자면 

1. SelectDataBasePath : DB 파일의 위치이다.
2. Query Data : 쿼리시 사용할 데이터 이다.
3. InitDataBase : DB 초기화 과정이다.
4. SelectData : DB에서 Data를 가져와 보여준다.
5. InsertData : DB에 Data를 넣는다.
6. FindData : 특정 Data 를 찾는다.
7. UpdateData : Data를 업데이트한다.
8. 버튼 옆의 상자 : 쿼리한 데이터를 표시한다.
9. 하단의 상자 : 간단한 로그를 보여준다.

그러면 코드로 들어가보자.  
참고로 하단의 상자에 로그를 적는내용은 빼고 적었다.

일단 DB를 선언한다.
```csharp
private static LiteDatabase database; // DB 선언
private static LiteCollection<User> userCollection; // 데이터를 담을 컬렉션 선언
```

다음으로는 DB와 연결을 하는 코드이다.  
InitDataBase 버튼 클릭시 호출된다.  

```csharp
private void ButtonInitDataBase_Click(object sender, EventArgs e)
{

    database = new LiteDatabase(textBoxDataBasePath.Text); // 지정된 경로로 초기화
    userCollection = database.GetCollection<User>("users");
}
```

다음은 데이터를 가져오는 부분이다.  
SelectData 버튼 클릭시 호출된다.  

```csharp
private void ButtonSelectData_Click(object sender, EventArgs e)
{
    listBoxSelectData.Items.Clear();
    var userList = userCollection.FindAll().ToList();

    foreach (var user in userList)
    {
        listBoxSelectData.Items.Add($"{user.Id}, {user.Password}, {user.Name}"); // ListBox 에 출력한다.
    }
}
```

데이터를 넣는 작업이다.  
데이터는 Query Data 에 , 로 구분하여 적어서 이용한다.   
InsertData 버튼 클릭시 호출된다.  

```csharp
private void ButtonInsertData_Click(object sender, EventArgs e)
{
    var tmpData = textBoxQueryData.Text.Split(','); // 데이터는 , 으로 구분한다.

    if (tmpData.Length == 3) // 데이터가 3개 인지 확인한다.
    {
        userCollection.Insert(new User()
        {
            Id = tmpData[0],
            Password = tmpData[1],
            Name = tmpData[2]
        }); // 데이터를 넣어준다.
    }
    else
    {
        MessageBox.Show("잘못된 데이터 입니다."); // 3개가 아니면 잘못된 입력이다.
    }
}

```

FindData 부분은 간단하게 쿼리문을 분석할 수 있는 함수를 만들어서 사용했다.  

```csharp
private void ButtonFindData_Click(object sender, EventArgs e)
{
    listBoxSelectData.Items.Clear();
    var userList = userCollection.Find(ParserQuery(textBoxQueryData.Text)).ToList();

    foreach (var user in userList)
    {
        listBoxSelectData.Items.Add($"{user.Id}, {user.Password}, {user.Name}");
    }
}

private Query ParserQuery(string data)
{
    Query tmpQuery = default;
    var tmpData = data.Split(' ');
    switch (tmpData[0].ToUpper())
    {
        case "EQ":
            tmpQuery = Query.EQ(tmpData[1], tmpData[2]);
            break;
        case "GTE":
            tmpQuery = Query.GTE(tmpData[1], tmpData[2]);
            break;
        case "GT":
            tmpQuery = Query.GT(tmpData[1], tmpData[2]);
            break;
        case "LT":
            tmpQuery = Query.LT(tmpData[1], tmpData[2]);
            break;
        case "LTE":
            tmpQuery = Query.LTE(tmpData[1], tmpData[2]);
            break;
        case "BETWEEN":
            tmpQuery = Query.Between(tmpData[1], tmpData[2], tmpData[3]);
            break;
        case "IN":
            tmpQuery = Query.In(tmpData[1], tmpData[2]);
            break;
        case "NOT":
            tmpQuery = Query.Not(tmpData[1], tmpData[2]);
            break;
        case "STARTWITH":
            tmpQuery = Query.StartsWith(tmpData[1], tmpData[2]);
            break;
        case "CONTAINS":
            tmpQuery = Query.Contains(tmpData[1], tmpData[2]);
            break;
        case "ALL":
            tmpQuery = Query.All(tmpData[1]);
            break;
    }

    return tmpQuery;
}
```

UpdateData 부분은 InsertData와 매우 비슷하다.  
다른점이 있다고 하면 Insert 가 Update 로 변경되었다.  

```csharp
private void ButtonUpdateData_Click(object sender, EventArgs e)
{
    var tmpData = textBoxQueryData.Text.Split(','); // 데이터는 , 으로 구분한다.

    if (tmpData.Length == 3) // 데이터가 3개 인지 확인한다.
    {
        userCollection.Update(new User()
        {
            Id = tmpData[0],
            Password = tmpData[1],
            Name = tmpData[2]
        }); // 데이터를 넣어준다.
    }
    else
    {
        MessageBox.Show("잘못된 데이터 입니다."); // 3개가 아니면 잘못된 입력이다
    }
}
```

이러면 프로그램은 완성이다. 
더 자세한 기능은 [LiteDB 깃허브](https://github.com/mbdavid/LiteDB/wiki/Getting-Started) 에서 볼 수 있다.


위에서 만든 프로그램의 소스코드이다.
[소스코드(github))]()

다른 궁금한 점이 있으면 댓글로 남겨두면 아는 범위 안에서 대답하겠다.