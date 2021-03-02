## 회원 관리 예제


### 회원 서비스 개발

```java
public Long join(Member member){
    memberRepository.findByName(member.getName())
        .ifPresent(m -> {
            throw new IllegalStateException("이미 존재하는 회원입니다");
        });

    memberRepository.save(member);
    return member.getId();
}
```
* ```ifPresent()``` : null이 아닌 어떠한 값이 있다면
* ```findByName```의 반환형이 Optional이기에 이를 통해 ifPresent 같은 메서드를 사용할 수 있다.
* 위의 경우 extract method로 따로 뽑아주는 것이 좋다.

<br>

**Extract Method 후**
```java
public Long join(Member member){
        //Ctrl + Alt + Shift + T refactoring -> extract method
        validateDuplicateMember(member);

        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
            .ifPresent(m -> {
                throw new IllegalStateException("이미 존재하는 회원입니다.");
            });
    }
```
<br><br>

**회원 서비스**

```java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;

import java.util.List;
import java.util.Optional;

public class MemberService {
    private final MemberRepository memberRepository = new MemoryMemberRepository();

    /**
     * 회원 가입
     */
    public Long join(Member member){
        validateDuplicateMember(member);

        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
            .ifPresent(m -> {
                throw new IllegalStateException("이미 존재하는 회원입니다.");
            });
    }

    /**
     * 전체 회원 조회
     */
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId){
        return memberRepository.findById(memberId);
    }

}
```

<br>

---

<br>

## 회원 서비스 테스트

만든 서비스 클래스에서 Ctrl + Shift + T -> Create New Test -> 모두 선택하고 ok하면

<br> 아래와 같이 껍데기가 만들어진다.

```java
package hello.hellospring.service;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class MemberServiceTest {

    @Test
    void join() {
    }

    @Test
    void findMembers() {
    }

    @Test
    void findOne() {
    }
}
```

<br>

테스트는 과감하게 메서드 명을 한글로 바꿔도 된다.

```java
class MemberServiceTest {

    //회원가입 위해
    MemberService memberService = new MemberService();

    @Test
    void 회원가입() {
    }

    @Test
    public void 중복회원예외() {
    }
    
    ...

}
```
<br>

테스트 시 ```given, when, then``` 주석을 써놓고 짜보자.

<br>

**일반적인 회원가입의 경우**
```java
    @Test
    void 회원가입() {
        //given
        Member member = new Member();
        member.setName("spring");

        //when
        Long saveId = memberService.join(member);

        //then
        Member findMember = memberService.findOne(saveId).get();
        //junit말고 assertj의 Assertions 사용
        //Assertions를 static import
        assertThat(member.getName()).isEqualTo(findMember.getName());

    }


```
<br>

**회원가입에서 예외가 잘 터지는지**

```java
    @Test
    public void 중복회원예외(){
        //given
        Member member1 = new Member();
        member1.setName("spring");

        Member member2 = new Member();
        member2.setName("spring");

        //when
        memberService.join(member1);
        /*
        {  member2를 join할 때 예외가 터져야 하는 부분  }
        */

        //then

    }
```

<br>

**예외가 잘 터지는가? 방법 1**
```java
        try{
            memberService.join(member2);

            //catch로 안가고 예외가 안터지면
            fail();            

        } catch(IllegalStateException e) {
            // 예외 메시지 동일한지 검증
            assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
        }

```

<br>

**예외가 잘 터지는가? 방법 2**
```java
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
```
* ```assertThrows``` : 뒤쪽 람다함수를 실행하면 앞쪽 명시된 exception이 터져야 함

<br><br>

  테스트 메서드가 끝날 때마다 저장소를 깔끔하게 비워줘야 한다고 전에 이야기 했다.
```java
    MemoryMemerRepository memberRepository = new MemoryMemberRepository();

    @AfterEach
    public void afterEach(){
        memberRepository.clearStore();
    }
```
<br><br>

여기까지 보면 Test와 MemberService에서 각각 ```new MemeoryMemRepository()```를 하게 되는데,
* 두 저장소를 각기 쓸 이유가 없고
* new로 다른 객체 생성하게 되면 내용물이 달라질 우려가 있다.
  즉 Test와 MemberService의 MemoryMemberRepository가 서로 다르다.
  지금은 Map에 static이 있어서 문제가 없지만 static을 없애면 각기 다른 DB가 되면서 문제가 생긴다.

<br>

  *같은 인스턴스 사용하게끔 하기 위해서 **생성자**를 통해 MemberService class에서 리포지토리를 외부로부터 넣어주게끔 수정한다.*

<br>

**수정 전**
```java
public class MemberService {
    private final MemberRepository memberRepository = new MemoryMemberRepository();
```

<br>

**수정 후**
```java
public class MemberService {
    private final MemberRepository memberRepository;

    //생성자로 외부에서 Repository 넣어줌
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

<br> 그리고 테스트 코드에서는


**수정 전**
```java

class MemberServiceTest {

    MemberService memberService = new MemberService();
    MemoryMemberRepository memberRepository = new MemoryMemberRepository();
```
<br>

**수정 후**
```java

class MemberServiceTest {

    MemberService memberService;
    MemoryMemberRepository memberRepository;

    @BeforeEach
    public void beforeEach() {
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }
}
```
* 각 테스트 실행 전 Repository를 만들고 memberService에다 넣어준다.
* 이렇게 하면 같은 repo를 사용하게 된다.
   MemberService 입장에서 본인이 직접 new 하지 않고 외부에서 넣어준다.

   이를 **Dependency Injection** 이라고 한다.

---

**최종코드 - MemberService**
```java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;

import java.util.List;
import java.util.Optional;

public class MemberService {
    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
         this.memberRepository = memberRepository;
    }

    /**
     * 회원 가입
     */
    public Long join(Member member){
        validateDuplicateMember(member);

        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
            .ifPresent(m -> {
                throw new IllegalStateException("이미 존재하는 회원입니다.");
            });
    }

    /**
     * 전체 회원 조회
     */
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId){
        return memberRepository.findById(memberId);
    }

}
```

<br><br>



**최종코드 - 테스트**
```java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemoryMemberRepository;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;

class MemberServiceTest {

    MemberService memberService;

    //for clear
    MemoryMemberRepository memberRepository;

    @BeforeEach
    public void beforeEach(){

        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }

    @AfterEach
    public void afterEach(){
        memberRepository.clearStore();
    }

    @Test
    void 회원가입() {
        //given
        Member member = new Member();
        member.setName("spring");

        //when
        Long saveId = memberService.join(member);


        //then
        Member findMember = memberService.findOne(saveId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());

    }

    @Test
    public void 중복회원예외(){
        //given
        Member member1 = new Member();
        member1.setName("spring");

        Member member2 = new Member();
        member2.setName("spring");


        //when
        memberService.join(member1);
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");

        //then
    }

    void findMembers() {
    }

    @Test
    void findOne() {
    }
}
```