# Test Framework

## JUnit & TestNG

|         |     JUnit     |     TestNG     |
|---------|:-------------:|:---------------|
|标记测试 |@Test          |@Test           |
|\<suite>内所有测试前执行|NA|@BeforeSuite|
|\<suite>内所有测试后执行|NA|@AfterSuite|
|\<test>内所有测试前执行|NA|@BeforeTest|
|\<test>内所有测试后执行|NA|@AfterTest|
|\<groups>内所有测试前执行|NA|@BeforeGroups|
|\<groups>内所有测试前执行|NA|@AfterGroups|
|测试类第一个测试前执行|@BeforeClass|@BeforeClass|
|测试类最后一个测试后执行|@AfterClass|@AfterClass|
|每个测试前执行|@Before|@BeforeMethod|
|每个测试后执行|@After|@AfterMethod|
|忽略测试|@Ignore|@Test(enabled=false)|
|预期异常|@Test(expected=)|@Test(expectedExceptions=)|
|超时|@Test(timeout=)|@Test(timeout=)|
|依赖|@Order|@Test(dependsOnMethods={})|
|参数化测试|@Parameters|@Parameters, @DataProvider|

_旧版本JUnit测试类必须继承TestCase，测试方法必须以test开头。每个测试方法前执行需复写setUp方法，每个测试方法后清理需复写tearDown方法。_  
_JUnit要求带@BeforeClass和@AfterClass的方法必须是静态方法；JUnit本身理念是测试互相隔离，测试依赖可以部分地通过使用@Order注解解决。_  
_JUnit要求带@Parameter必须是静态方法，同时返回类型必须是List[]。_  
_TestNG支持JUnit注解，只需要在testng.xml将使用JUnit注解的类放在test元素下，并将属性"junit"设置为"true"即可。TestNG支持java原生的assert。_

TestNG执行流程
```flow
s=>start: 开始
e=>end: 结束
bs=>operation: @BeforeSuite
bt=>operation: @BeforeTest
bc=>operation: @BeforeClass
bg=>operation: @BeforeGroups
bm=>operation: @BeforeMethod
t=>operation: @Test
nt=>condition: 下一个测试
am=>operation: @AfterMethod
ag=>operation: @AfterGroups
ac=>operation: @AfterClass
at=>operation: @AfterTest
as=>operation: @AfterSuite
s->bs->bt->bc->bg->bm->t->am->nt
nt(yes,存在)->bm
nt(no)->ag->ac->at->as->e
```

-----------------------------------------------------------------------------------------------------------------------------

## EasyMock & Mockito

|         |     EasyMock     |     Mockito     |
|---------|:----------------:|:----------------|
|mock     |@Mock             |@Mock            |
|部分mock| NA|@Spy|
|测试对象|@TestSubject|@InjectMocks|


EasyMock执行流程
```flow
s=>start: @RunWith(EasyMockRunner.class)
e=>end: Assert.assert*
m=>operation: @Mock/EasyMock.mock
ts=>operation: @TestSubject/testSubject.addListener(mock)
t=>subroutine: @Test|test
when=>operation: EasyMock.expect|test
then=>operation: andReturn/andThrow/;EasyMock.expectLastCall|test
times=>operation: anyTimes, times, atLeastOnce|test
replay=>operation: EasyMock.replay|test
test=>operation: testSubject.exec|test
verify=>operation: EasyMock.verify|test

s->m->ts->when->then->times->replay->test->verify->e
```

_EasyMock要求被注入的测试类必须手动生成测试对象，Mockito无论是Mock还是Spy都可自动生成对象（类型是接口或抽象类时仍需手动生成）。_   
_EasyMock严格按照replay、verify流程，Mockito无需replay。_  
_EasyMock不支持部分mock。_  
_EasyMock和Mockito都支持任意参数的mock，同时都要求同一个方法只要有一个是任意参数时，其它参数也必须是任意参数，而不能指定。_

```java
@RunWith(MockitoJUnitRunner.class)//MockitoAnnotations.initMocks(this);
public class MockTest {

    @InjectMocks
    private TestSubject ts;// = new TestSubject();
    
    @Spy
    private InjectableMockableObject imo;
    //@Spy
    //@InjectMocks
    //private InjectableMockableObject imo = new InjectableMockableObject();
    
    @Mock
    private MockableObject mo;
    
    @Test
    public void testOriginMethodName() {
        Mockito.when(mo.doX(Mockito.any())).thenReturn(1);
	//imo.doY(..);
	Mockito.doNothing().when(imo).doZ(Mockito.anyString(), Mockito.any());
	Assert.assertTrue(ts.exec());
    }
}
```

-----------------------------------------------------------------------------------------------------------------------------

## PowerMock & Jmockit

由于EasyMock和Mockito不能对private、static、final方法进行mock，这时候就需要使用PowerMock或者Jmockit了。	 

|         |     PowerMock     |     Jmockit     |
|---------|:----------------:|----------------:|
|修改类及其父类|@PrepareForTest|MockUp|
|仅修改类|@PrepareOnlyThisForTest|MockUp|
|runner|PowerMockRunner|JMockit|
|static|verifyStatic/mockStatic|Expectations{{Klass.static}}|
|private|verifyPrivate/when(mock, privateMthodName, args)|Expectations{{Deencapsulation.invoke(mock, privateMthodName, args)}}|
|final|when|Expectations{{mock.finalMethod}}|
|native|when|Expectations{{mock.nativeMethod}}|
|constructor|verifyNew/whenNew|Expectations{{Deencapsulation.newInstance}}|

_JMockit的@Mocked类似于EasyMock或Mockito的Mock，JMockit的@Injectable类似于Mockito的@Spy。_  
_JMockit定义类一系列的any*在Invocations中，同时还有其它protected变量，比如result和times。_  
_JMockit的Deencapsulation类封装了一系列对字段、方法的操作。_

```java
@RunWith(PowerMockRunner.class)
@PrepareOnlyThisForTest(CannotOverride.class)
public class XxxTest {

    @InjectMocks
    private TestSubject testSubject;
	
    @Test
    public void testXyz() {
	PowerMockito.mockStatic(CannotOverride.class);
	PowerMockito.when(CannotOverride.staticMethod()).thenReturn(null);
	testSubject.exec(params);
	//Assert.assert*
    }
    
}
```

```java
@RunWith(JMockit.class)
public class XxooTest {

    @Tested
    private TestSubject testSubject;

    @Test
    public void testAbc() {
	final string expect = "expectationResult";
	new Expectations(){
	    {
	        Klazz.staticMethod(any);
		result = expect;
	    }
	};
	Assert.assertEquals(expect, testSubject.exec(params));
    }

}
```

-----------------------------------------------------------------------------------------------------------------------------

## Spring Test

1. Spring bean

```java
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {AppConfig.class})/* locations = {"/applicationContext.xml"} */
public class SpringTest {

    @Resource
    private SpringBean bean;

    @Test
    @Rollback
    public void testBeanMethod() {
        String ms = System.currentTimeMillis() + "";
        Assert.assertEquals(ms, bean.echo(ms));
    }
}
```

2. Spring MVC

```java
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import javax.annotation.Resource;

@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = {"file:src/main/webapps/WEB-INF/applicationContext.xml", "file:src/main/webapps/WEB-INF/dispatcher-servlet.xml"})
@ActiveProfiles("dev")
public class SpringMvcTest {

    @Resource
    private WebApplicationContext wac;

    private MockMvc mvc;

    @Before
    public void setUp() {
        mvc = MockMvcBuilders.webAppContextSetup(wac).build();/* MockMvcBuilders.standaloneSetup(new SpringController()).build(); */
    }

    @Test
    @Rollback
    public void testController() throws Exception{
        String me = "iMinusMinus";
        Assert.assertEquals(me, mvc.perform(MockMvcRequestBuilders.get("/" + me)).andReturn().getResponse().getContentAsString());
    }
}
```

3. Spring Boot

```java
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

import javax.annotation.Resource;
import java.util.Collections;

@RunWith(SpringRunner.class)
@WebMvcTest//@SpringBootTest
@ContextConfiguration(classes = AppConfig.class)
public class SpringBootMvcTest {

    @Resource
    private MockMvc mockMvc;

    @MockBean
    private SpringService mockService;

    @Test
    public void testSpringBoot() throws Exception {
        Mockito.when(mockService.mockFun(Mockito.<MockDTO>any())).thenReturn("OK");
        MvcResult result = mockMvc.perform(MockMvcRequestBuilders.post("/path").contentType("application/json").content("{}")).andReturn();
        Assert.assertTrue(result.getResponse().getContentAsString() == "OK");
    }
}
```
