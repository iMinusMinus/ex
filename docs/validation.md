校验
======
##曾经
> 自己动手，丰衣足食

    public Response entry(Request req) {
        boolean valid = validate(req);
        if(!valid) {
    		throw new ValidationException();//type 1
			/* type 2
			return new Response(ResultCode.INVALID_PARAMETERS);
			*/
		}
        //其它代码		
	}
	
	private boolean validate(Request req) {
	    if(req.getField1() == null) {
		    return false;
		}
		if(field.getField2() < 0 || field.getField2() > 10) {
			return false;
		}
		// 其它字段各种校验
		return true;
	}
>> 余额代偿系统	
	![za-fcp-prebiz-yedc](https://github.com/iMinusMinus/ex/blob/master/images/validation/yedc.png?raw=true)
>> 马上花系统	
	![creditpre-retail](https://github.com/iMinusMinus/ex/blob/master/images/validation/msh1.png?raw=true)
	![creditpre-retail](https://github.com/iMinusMinus/ex/blob/master/images/validation/msh2.png?raw=true)
>> 保险某系统
	![za-ccp-insurance-share](https://github.com/iMinusMinus/ex/blob/master/images/validation/policy1.png?raw=true)
	![za-ccp-insurance-share](https://github.com/iMinusMinus/ex/blob/master/images/validation/policy2.png?raw=true)
	![za-ccp-insurance-share](https://github.com/iMinusMinus/ex/blob/master/images/validation/policy3.png?raw=true)
> 不会驾驭轮子的程序员不是好司机

>>	
    // Google方式
	private boolean validate(Request req) {
		try {
			Preconditions.checkNotNull(req.getField());
			//其它字段各种校验
			return true;
		} catch(Exception e) {
			return false;
		}	
	}
>>	
    //Spring方式
	private boolean validate(Errors error, Request req) {
		ValidationUtils.rejectIfEmpty(error, "fieldName", "{classFullQualifiedName.fieldName.ConstraintType}", "XX不能为空");
		ValidationUtils.invokeValidator(new RequestValidator(), req, error);
		return error.hasError();
	}
>>    
	public static class RequestValidator implements org.springframework.validation.Validator {
>>	
		public boolean supports(Class<?> clazz) {
			return Request.class.isAssignableFrom(clazz);
		}
>>		
		public void validate(Object target, Errors errors) {
			//校验字段
		}
	}
-----------------------------------------------------------------------------------------------------------------------
__ 遇到的问题 __

1. 分组校验。

问：比如余额代偿系统马上金和马上还都在使用，但某个参数只有马上金需要做校验。

答：子类覆写父类校验，马上金和马上还各自做私有校验，再调用父类校验。

问：马上金接入直营APP和H5，而某些字段只有H5需要做校验。

答：拆分子类，或判断来源做分支校验。

-----------------------------------------------------------------------------------------------------------------------

##标准

   请参考[Bean Validation]
   [Bean Validation]: http://beanvalidation.org/specification/ "Specification hosted on Red Hat"
   
1. JSR 303 Bean Validation1.0

   @AssertFalse, @AssertTrue, @DecimalMax, @DecimalMin, @Digits, @Future, @Max, @Min, @NotNull, @Null, @Past, @Pattern, @Size
   
   message
   
   group
   
   @Valid
   
   @GroupSequence
   
   ![Bean](https://github.com/iMinusMinus/ex/blob/master/images/validation/bean.png?raw=true)
   ![校验示例](https://github.com/iMinusMinus/ex/blob/master/images/validation/test.png?raw=true)
   ![自定义注解](https://github.com/iMinusMinus/ex/blob/master/images/validation/constraint.png?raw=true)
   ![自定义注解验证器](https://github.com/iMinusMinus/ex/blob/master/images/validation/constraintValidator.png?raw=true)
   
2. JSR 349 Bean Validation1.1

    @ValidateOnExecution
    
	@SupportedValidationTarget
    
	@ConvertGroup
	
	EL
	
   ![](https://github.com/iMinusMinus/ex/blob/master/images/validation/crossContraint.png?raw=true "自定义注解")
   ![自定义验证器](https://github.com/iMinusMinus/ex/blob/master/images/validation/crossValidator.png?raw=true)
   ![跨参数校验](https://github.com/iMinusMinus/ex/blob/master/images/validation/crossTest.png?raw=true)
   ![综合：message, EL & ConvertGroup](https://github.com/iMinusMinus/ex/blob/master/images/validation/mix.png?raw=true)
	
3. JSR 380 Bean Validation2.0

	@Email, @NotEmpty, @NotBlank, @Positive, @PositiveOrZero, @Negative, @NegativeOrZero, @PastOrPresent, @FutureOrPresent
    
	List<@Positive Integer>
    
	@Repeatable
	
	![容器泛型注解](https://github.com/iMinusMinus/ex/blob/master/images/validation/generic.png?raw=true)
	
<table>
	<thead><tr><th>特性</th><th>Hibernate Validator</th><th>Apache BVal</th></tr></thead>
	<tbody>
		<tr><td>Annotation</td><td>&#10004;</td><td>&#10004;</td></tr>
		<tr><td>xml</td><td>&#10004;</td><td>&#10004;</td></tr>
		<tr><td>1.0</td><td>Hibernat Validator 4.x</td><td>bval-jsr303</td></tr>
		<tr><td>1.1</td><td>Hibernat Validator 5.x</td><td>bval-jsr</td></tr>
		<tr><td>2.0</td><td>Hibernat Validator 6.x</td><td>&#10006;</td></tr>
	</tbody>
</table>

-----------------------------------------------------------------------------------------------------------------------


##应用

1.Spring MVC

	/**
	 * 使用代码示例
	 */
	@Controller
	@RequestMapping("/rest")
	public class RestController {
	
		@RequestMapping(value="/test")
		@ResponseBody
		//@ExceptionHandler(BindException.class)
		public AnyResponse(@Valid AnyRequest request, BindingResult br) {
			//如果request校验出不符合约束字段，br.hasError
		}
	
	}
	
原理：HandlerMapping --> HandlerExecutorChain --> HandlerAdapter.handle --> AnnotationMethodHandlerAdapter.invokeHandlerMethod --> HandlerMethodInvoker.resolveHandlerArguments	
   ![Spring MVC argument](https://github.com/iMinusMinus/ex/blob/master/images/validation/arg1.png?raw=true)
   ![Spring MVC data binder](https://github.com/iMinusMinus/ex/blob/master/images/validation/arg2.png?raw=true)
   ![Spring MVC bind](https://github.com/iMinusMinus/ex/blob/master/images/validation/bind.png?raw=true)
   ![Spring MVC validate](https://github.com/iMinusMinus/ex/blob/master/images/validation/validate.png?raw=true)
	
2.HSF AOP

	//在调用实际方法前进行校验，有违反约束则抛出异常
	@Aspectj
	@Component
	public class ValidationAspect {
	
	    @Resource
	    private Validator validator;
		
		@Pointcut("execution (* packageName.className.*(..))")
		private void pointcut() {
		}
		
		@Before(value = "pointcut() && args(request, ..)")
		public void advice(Object request) {
		    Set<?> violations = validator.validate(request, Default.class);
            if (!violations.isEmpty()) {
                throw new ValidationException();
            }
		}
		
	}

   ![AOP Validation](https://github.com/iMinusMinus/ex/blob/master/images/validation/aop.png?raw=true)
