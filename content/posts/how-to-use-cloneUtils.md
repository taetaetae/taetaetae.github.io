---
title: 자바 객체 복사하기 ( feat. how to use CloneUtils? )
date: 2018-08-21 18:02:47
categories:
  - tech
tags: 
  - cloneUtils
  - java deep copy
  - archives-2018
url : /2018/08/21/how-to-use-cloneUtils/
featuredImage: /images/how-to-use-cloneUtils/clone_java.jpg

---
자바(Java)로 개발을 하다보면 한번쯤 객체를 복사하는 로직을 작성할때가 있다. 그때마다 나오는 이야기인 `Shalldow Copy` 와 `Deep Copy`. 한국어로 표현하면 얕은 복사와 깊은 복사라고 이야기를 하는데 이 두 개념의 차이는 아주 간단하다. 객체의 주소값을 복사하는지, 아니면 객체의 실제 값(value)를 복사하는지. <!-- more --> 이 둘의 차이점을 소개하는 글들은 워낙 많으니 패스하도록 하고 이번 포스팅에서는 `Deep Copy`를 할때 `org.apache.http.client.utils` 하위에 있는 `CloneUtils` 사용법에 대해 정리 하고자 한다.

> 그냥 쓰면 되는거 아닌가? 라고 생각했지만 (별거 아니라고 생각했지만) 해보고 안해보고의 차이는 엄청컸고 사용할때 주의점이 몇가지 있어 정리 하려고 한다.

예제에 앞서 본 포스팅에서 사용할 객체를 간단히 정리하면 다음과 같다. (학교에서 학생 신상정보를 관리한다고 가정해보자.)

```java
public class Student {
	String name; // 이름
	int age; // 나이
	Family family; // 가족
}

public class Family {
	String name;  // 이름
	int age; // 나이
	boolean isOfficeWorkers; // 직장인 여부
}

public class PhysicalInformation {
	int height; // 키
	int weight; // 몸무게
}
```

### 객체는 Cloneable interface 를 implement 해야하고 clone 메소드를 public 으로 override 해야한다.
당연한 이야기가 될수도 있으나 `CloneUtils`를 사용하기 위해서는 해당 객체는 Cloneable interface 를 implement 해야한다. 그리고 나서 clone 메소드를 override 해야되는데 여기서 가장 중요한점은 외부에서도 호출이 가능해야하기 때문에 `public` 으로 override를 해야한다. (기본은 protected 로 되어있다.) 우선 간단히 객체를 생성하고 출력부터 해보자. (출력을 이쁘게 하기 위해 `ToStringBuilder.reflectionToString`을 사용하였다.)

```java
PhysicalInformation physicalInformation = new PhysicalInformation();
physicalInformation.height = 180;
physicalInformation.weight = 70;

System.out.println(ToStringBuilder.reflectionToString(physicalInformation, ToStringStyle.DEFAULT_STYLE));
```
결과는 당연히 

```
PhysicalInformation@5d6f64b1[height=180,weight=70]
```

이제 Cloneable interface 를 implement 하고 clone 메소드를 public 으로 override 한뒤, CloneUtils를 사용해서 객체를 복사해보자. 테스트를 하면서 `Shalldow Copy`도 해보자.

```java
// class setting
public class PhysicalInformation implements Cloneable{
	int height;
	int weight;

	@Override
	public Object clone() throws CloneNotSupportedException { // public 으로 바꿔주자.
		return super.clone();
	}
}

// test code
PhysicalInformation physicalInformation = new PhysicalInformation();
physicalInformation.height = 180;
physicalInformation.weight = 70;

PhysicalInformation physicalInformationShalldowCopy = physicalInformation;
PhysicalInformation physicalInformationDeepCopy = null;
try {
	physicalInformationDeepCopy = (PhysicalInformation)CloneUtils.clone(physicalInformation);
} catch (CloneNotSupportedException e) {
	e.printStackTrace();
}

// 원본
System.out.println(ToStringBuilder.reflectionToString(physicalInformation, ToStringStyle.DEFAULT_STYLE));
// 얕은 복사
System.out.println(ToStringBuilder.reflectionToString(physicalInformationShalldowCopy, ToStringStyle.DEFAULT_STYLE));
// 깊은 복사
System.out.println(ToStringBuilder.reflectionToString(physicalInformationDeepCopy, ToStringStyle.DEFAULT_STYLE));

// 값 변경
physicalInformation.weight = 80;
physicalInformation.height = 170;

// 원본
System.out.println(ToStringBuilder.reflectionToString(physicalInformation, ToStringStyle.DEFAULT_STYLE));
// 얕은 복사
System.out.println(ToStringBuilder.reflectionToString(physicalInformationShalldowCopy, ToStringStyle.DEFAULT_STYLE));
// 깊은 복사
System.out.println(ToStringBuilder.reflectionToString(physicalInformationDeepCopy, ToStringStyle.DEFAULT_STYLE));
```

결과는 원본과 얕은 복사를 한것은 메모리 주소(?)가 같으나 깊은 복사를 한것은 데이터는 같지만 주소가 다르고 값을 변경해도 영향을 주지 않는다. (완전히 서로다른 객체인것을 증명)

```markdown
PhysicalInformation@1376c05c[height=180,weight=70]
PhysicalInformation@1376c05c[height=180,weight=70]
PhysicalInformation@1b4fb997[height=180,weight=70]

PhysicalInformation@1376c05c[height=170,weight=80]
PhysicalInformation@1376c05c[height=170,weight=80]
PhysicalInformation@1b4fb997[height=180,weight=70]
```

만약 위에서 clone을 기본값인 protected로 override를 하게 되면 어떤 결과를 가져올까?

```markdown
Exception in thread "main" java.lang.NoSuchMethodError: com.PhysicalInformation.clone()
	at org.apache.http.client.utils.CloneUtils.cloneObject(CloneUtils.java:55)
	at org.apache.http.client.utils.CloneUtils.clone(CloneUtils.java:77)
	at com.Test.main(Test.java:16)

```

접근제한자에서 눈치를 챌수도 있었겠지만 접근을 할수없어 CloneUtils 이 리플렉션을 하는 과정에서 Exception을 발생한다. 꼭! public 으로 override를 해주자.

### 객체 내에 clone이 안되는 변수는 별도 처리가 필요하다.
객체 내에 있는 멤버 변수는 원시 변수(int, char, float 등) , Immutable Class (String, Boolean, Integer 등) 또는 Enum 형식일 때는 원본의 값을 바로 대입해도 되지만, 그렇지 않을 때는 멤버변수의 clone을 호출하여 복사해야 한다. 말로만 보면 무슨이야기 인지 모르니 예제를 보자.

```java
public class Student implements Cloneable {
	String name;
	int age;
	Family family;

	@Override
	public Object clone() throws CloneNotSupportedException {
		return super.clone();
	}
}
```

Student 클래스에서 Cloneable 를 implements 하고 clone 메소드를 override 하였다. (여기서 구멍이 있다!!) 그다음 Family 클래스는 초기 그대로 두고 CloneUtils을 사용해서 객체를 복사하는 코드를 작성해보자.

```java
Student student = new Student();
student.name = "taetaetae";
student.age = 20;

Family family = new Family();
family.age = 25;
family.isOfficeWorkers = true;
family.name = "son";
student.family = family;

Student studentDeepCopy = null;
try {
	studentDeepCopy = (Student)CloneUtils.clone(student);
} catch (CloneNotSupportedException e) {
	e.printStackTrace();
}
// student 객체 복사여부 확인
System.out.println(ToStringBuilder.reflectionToString(student, ToStringStyle.DEFAULT_STYLE));
System.out.println(ToStringBuilder.reflectionToString(studentDeepCopy, ToStringStyle.DEFAULT_STYLE));
// student 내 family 객체 복사 여부 확인
System.out.println(ToStringBuilder.reflectionToString(student.family, ToStringStyle.DEFAULT_STYLE));
System.out.println(ToStringBuilder.reflectionToString(studentDeepCopy.family, ToStringStyle.DEFAULT_STYLE));
```
Student 객체 안에 int, String, 그리고 별도로 만든 객체인 Family 가 있는 상황에서 복사를 해보자. 결과는 어떻게 나왔을까?

```markdown
com.Student@1376c05c[name=taetaetae,age=20,family=com.Family@51521cc1]
com.Student@deb6432[name=taetaetae,age=20,family=com.Family@51521cc1]

com.Family@51521cc1[name=son,age=25,isOfficeWorkers=true]
com.Family@51521cc1[name=son,age=25,isOfficeWorkers=true]
```
위에서 말했던 구멍의 결과를 볼수 있다. Student 객체는 주소값이 다른걸 보니 깊은 복사가 되었지만 그 안에 있는 Family 형 변수는 얕은 복사가 된것을 확인할수 있다.

위에서 말한것과 같이 clone이 안되는 경우는 (다시 말하면 원시변수나 Immutable Class, enum 등 clone을 지원하는 객체가 아닐경우) 별도로 clone이 되도록 설정이 필요하다.

그래서 Family 도 Cloneable 를 implements 하고 clone 메소드를 override 해준다음 최상위 객체였던 Student의 clone 메소드를 아래처럼 조금 수정해주고 나서 테스트를 해보면 이쁘게 깊은 복사가 된것을 확인할수 있다.

```java
// 복사가 될수있도록 설정
public class Family implements Cloneable{
	String name;
	int age;
	boolean isOfficeWorkers;

	@Override
	public Object clone() throws CloneNotSupportedException {
		return super.clone();
	}
}

// 일단 복사를 하고, 멤버 객체 자체를 복사한 다음 대입 
public class Student implements Cloneable {
	String name;
	int age;
	Family family;

	@Override
	public Object clone() throws CloneNotSupportedException {
		Student student = (Student)super.clone();
		student.family = (Family)CloneUtils.clone(student.family);
		return student;
	}
}

```

그러고 테스트를 해보면 내부 멤버 변수도 복사가 된것을 확인할수 있다.

```markdown
com.Student@51521cc1[name=taetaetae,age=20,family=com.Family@1b4fb997]
com.Student@28ba21f3[name=taetaetae,age=20,family=com.Family@694f9431]
com.Family@1b4fb997[name=son,age=25,isOfficeWorkers=true]
com.Family@694f9431[name=son,age=25,isOfficeWorkers=true]
```

### 마치며
멤버변수가 List, Map, Set 등 여러 유형으로 복잡하게 만들어졌을경우 각각 복사를 해주는 등 다양한 케이스에 유연하게 대응할수 있어야 하겠다. 
객체복사? 그거 그냥 하면 되는거 아냐? 라고 볼수도 있으나 입개발 하는 사람과 직접 구현해보고 내부 코드까지 들여다보는 수고를 하는 사람의 차이는 언젠간 분명히 드러날꺼라 생각한다. (필자가 그랬으니ㅠㅠ)