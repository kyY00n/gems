# Optional의 map을 이럴 때 쓰면되는구나

&#x20;

미션을 진행하면서 데이터베이스로부터 조회해서 도메인 객체를 만드는 과정에서 Dao와 Repository를 사용했는데 그 과정에서 Optional을 사용해서 안전하게 데이터를 가져오려고 했다.

그런데 entity를 다루는 dao, 도메인 객체를 다루는 repository 두 계층에서 모두 optional을 사용하려다보니 형변환이 필요했다.

Optional을 이론적으로 공부하긴 했지만 다음과같은 코드가 탄생했다 ^^

```java
public Optional<Line> findById(Long id) {
    Optional<LineEntity> optionalLineEntity = lineDao.findById(id);
    if (optionalLineEntity.isEmpty()) {
        return Optional.empty();
    }

    LineEntity lineEntity = optionalLineEntity.get();

    return Optional.of(convertToDomain(lineEntity));
}
```

코드를 짜면서도 이렇게 쓰는 건 아닌 것 같아서 미션끝나고 전에 정리했던 내용을 찾아보았다.

### Optional.map

Optional을 그대로 사용하고싶지만 타입을 변경하고 싶은 경우에 (지금 내 코드) 사용할 수 있는 api 다.

다음은 Optional의 map을 스트림과 비교한 그림인데, 기억하기에 도움이 되는 것 같아 가져왔다.

![](https://github.com/kyY00n/gems/assets/61582017/4c992715-e192-47c1-95db-025dc1ac2923)

출처: <모던 자바 인 액션>

### Optional.flatMap

flatMap이라는 녀석도 있다. 만약 우리가 map을 한이 옵셔널이라면 어떻게 될까? 즉 그림과 같은 상황이다.

![](https://github.com/kyY00n/gems/assets/61582017/4c5a829b-6307-4b20-8b6e-6e1047e216a6)

map이 된 타입이 세모를 감싸는 Optional인 것이다. 이경우에 mapping을 해주기 위해서 Optional을 풀어주는 과정이 필요한데, 이를 함께 해주는 것이 flatMap이다.

다음같은 경우 사용할 수 있겠다.

Person 예제

```java
public class Person {
    private Optional<Car> car; // 사람이 차를 소유했을 수도 하지 않았을 수도 있으므로 Optional
    public Optional<Car> getCar() { return car; }
}
public class Car {
    private Optional<Insurance> insurance; // 자동차 보험에 가입 or 미가입일수 있으므로
    public Optional<Insurance> getInsurance() { return insurance; }
}
public class Insurance {
    private String name; // 보험회사에는 반드시 이름이 존재함 
    public String getName() { return name; }
}
```

```java

public String getCarInsuranceName(Optional<Person> person) { 
    return person.flatMap(Person::getCar) // Optional<Person>를 Optional<Car>로 반환
                   .flatMap(Car::getInsurance) // Optional<Car>를 Optional<Insurance>로 반환
                   .map(Insurance::getName) // getName은 String을 반환하므로 flatMap 필요 X 
                   .orElse("Unknown"); // 결과 Optional이 비어있으면 기본값 사용 
}
```
