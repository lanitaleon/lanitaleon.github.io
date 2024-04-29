---
layout: post
title: SpringBoot + Mockito + JUnit5 Demo
---

# Dependency

spring-boot-starter-test 2.6.6

其中内置了：

```shell
junit-jupiter 5.8.2
mockito-core 4.0.0
mockito-junit-jupiter 4.0.0
```

除此之外，还需要引入 mockito-inline 4.0.0，用于模拟静态方法；

注：

本节 demo 用的 spring-boot 版本比较古早，如果是新版 mockito-core 就不需要额外引入 mockito-inline。

# Overview

先看一个简单的单元测试类需要哪些基本元素。

```java
public class User {
	private Long id;
	private String name;
}

public interface UserService {
	String updateUserName(Long id, String name);
}

public UserServiceImpl implements UserService {
	@Resource
	private UserRepo userRepo;

	@Override
	String updateUserName(Long id, String name) {
		User user = userRepo.findById(id);
		// ...
		return user.updateName(name);
	}
}

@ExtendWith(MockitoExtension.class)
class UserTests {
	@Mock
	private UserRepo userRepo;
	@InjectMocks
	private UserServiceImpl userServiceImpl;

	@Test
	void updateUserName() {
		User mockedUser = mock(User.class);
		when(userRepo.findById(1L)).thenReturn(mockedUser);
		when(mockedUser.updateName(any(String.class))).thenReturn("new name");
		Assertions.assertEquals("new name", userServiceImpl.updateUserName(1L, "new name"));
	}
}

```

`@ExtendWith(MockitoExtension.class)`

单元测试类需要附加 `mockito` 扩展类；

`@InjectMocks`

引入测试对象；

注意：接口类不需要测试，所以直接编写实现类的单元测试；

`@Mock` 

测试对象中引入的其他对象；

`mock(XXX.class)` 

虚拟一个类对象；

`when(XXX.function(any(YYY.class))).thenReturn(...);` 

模拟一个方法的执行和返回值，即打桩；

`Assertions` 

比对预期输出，常用方法：`assertEquals`，`assertTrue`，`assertThrows`，`assertDoesNotThrow` 等；

# Functions

## any

模拟各种参数，`mockito` 提供了非常多的实现，在此不一一罗列。

```java
import org.mockito.ArgumentMatchers.*;

any(UpdateUserParams.class)
anyInt();
anyString();
anyBoolean()
// ...
```

需要注意的是，如果一个参数是真实的，那么其他参数也被要求是真实的。

```java
public interface UserService {
	Long updateAddress(AddressParams addressParams, RequestOptions requestOptions);
}

@Test
void updateAddress() {
	// 两个都是 any 是可以的
	when(userServiceImpl.updateAddress(any(AddressParams.class), any(RequestOptions.class)))
		.thenReturn(1L);

	// 这种情况是不可以的，会抛出异常 
	// 因为第一个参数是虚的，第二个参数是实的
	RequestOptions requestOptions = new RequestOptions("apiKey");
	when(userServiceImpl.updateAddress(any(AddressParams.class), requestOptions))
		.thenReturn(1L);		
}

```

## when + answer

`thenReturn` 倾向于返回一个确定的值，比如 `1L`, `"A"`;

`thenAnswer` 倾向于返回一个动态构造的值，比如一个实例对象；

```java
public class Order {
	private String status;
}

public interface OrderService {
	Order create();
}

public interface FlowsService {
	Order finalizeOrder(Order order);
}

public class OrderServiceImpl implements OrderService {
	@Resource
	private FlowsService flowService;

	@Override
	public Order create() {
		// ...
		flowService.finalizeOrder(Order order);
		// ...
		return order;
	}
}

@ExtendWith(MockitoExtension.class)
class OrderTests {
	@Mock
	private FlowsService flowService;
	@InjectMocks
	private OrderServiceImpl orderServiceImpl;

	@Test
	void finalizeOrder() {
		Order order = new Order("status_draft");
		when(flowService.finalizeOrder(any(Order.class)))
			.thenAnswer(v -> {
				order.setStatus("status_open");
				return order;
			});
		Assertions.assertEquals("status_open", orderService.create().getStatus());
	}
}
```

## when + throw

模拟抛出异常；

```java
@Test
void createUser() {
	when(validateServce.validate(any(User.class))).thenThrow(InvalidUserException.class);
	Assertions.assertThrows(InvalidUserException.class, 
		() -> userServiceImpl.updateUserName(1L, "name"));
}
```

## mockStatic + when

模拟静态函数；

```java
public class User {
	// ...

	public static Long create(UserCreateParams params) {
		// ...
		return id;
	}
}

@Test
void createUser() {
	try (MockedStatic<User> methods = mockStatic(User.class)) {
		methods.when(() -> User.create(any(UserCreateParams.class))).thenReturn(1L);
		Assertions.assertDoesNotThrow(() -> userServiceImpl.create(new UserCreateParams()));
	}
}
```

## doNothing + when

针对 `void` 方法，没有返回值的时候，需要用 `doNothing()` 进行模拟。

```java
@Test
void batchInsert() {
	List<User> users = List.of(...);
	doNothing().when(userRepo).batchInsert(users);
	Assertions.assertDoesNotThrow(() -> userServiceImpl.batchInsert(users));
}
```

## verify

校验方法是否被调用，调用次数和顺序是否符合预期。

```java
public class MyList extends AbstractList<String> {

    @Override
    public String get(final int index) {
        return null;
    }
    
    @Override
    public int size() {
        return 1;
    }
}

@Test
void testMyList() {
	// 校验 size() 被调用了
	List<String> mockedList = mock(MyList.class);
	mockedList.size();
	verify(mockedList).size();	
	
	// 校验 mockedList2 没有发生任何事
	List<String> mockedList2 = mock(MyList.class);
	verifyNoInteractions(mockedList2);

	// 校验 mockedList3 被调用了1次
	List<String> mockedList3 = mock(MyList.class);
	mockedList3.size();
	verify(mockedList3, times(1)).size();

	// 校验 mockedList4 的执行顺序
	List<String> mockedList4 = mock(MyList.class);
	mockedList4.size();
	mockedList4.add("a parameter");
	mockedList4.clear();
	InOrder inOrder = Mockito.inOrder(mockedList4);
	inOrder.verify(mockedList4).size();
	inOrder.verify(mockedList4).add("a parameter");
	inOrder.verify(mockedList4).clear();

	// 校验 mockedList5 的调用次数是否符合预期
	List<String> mockedList5 = mock(MyList.class);
	mockedList5.clear();
	mockedList5.clear();
	mockedList5.clear();
	verify(mockedList5, atLeast(1)).clear();
	verify(mockedList5, atMost(10)).clear();
}
```

## stubbing a spy

和 `mock()` 不同的是，`spy()` 的对象是真实的对象实例，方法也会直接调用真实对象的方法；

```java
@Test
void givenUsingSpyMethod_whenSpyingOnList_thenCorrect() {
    List<String> list = new ArrayList<String>();
    List<String> spyList = spy(list);

    spyList.add("one");
    spyList.add("two");

    verify(spyList).add("one");
    verify(spyList).add("two");

    assertThat(spyList).hasSize(2);
}
```

但是这并不意味着 `spy` 不能打桩……

```java
@Test
void givenASpy_whenStubbingTheBehaviour_thenCorrect() {
    List<String> list = new ArrayList<String>();
    List<String> spyList = spy(list);

    assertEquals(0, spyList.size());

    doReturn(100).when(spyList).size();
    assertThat(spyList).hasSize(100);
}
```

## Assertions

junit 提供了非常多的方法，在此不一一罗列。

```java
import org.junit.jupiter.api.Assertions.*;

assertThrows(SomeException.class, () -> yourServiceImpl.yourMethod());
assertDoesNotThrow(() -> yourServiceImpl.yourMethod());
assertTrue(yourServiceImpl.yourMethod());
assertEquals(yourServiceImpl.yourMethod());
// ...
```
