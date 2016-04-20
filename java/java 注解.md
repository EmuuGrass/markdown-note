##Java注解##
###为什么要引入注解？###
XML配置其实就是为了分离代码和配置而引入的。假如你想为应用设置很多的常量或参数，这种情况下，XML是一个很好的选择，因为它不会同特定的代码相连。如果你想把某个方法声明为服务，那么使用Annotation会更好一些，因为这种情况下需要注解和方法紧密耦合起来。

###Annotation是如何工作的？怎么编写自定义的Annotation？###
Annotations仅仅是元数据，和业务逻辑无关。理解起来有点困难，但就是这样。如果Annotations不包含业务逻辑，那么必须有人来实现这些逻辑。元数据的用户来做这个事情。Annotations仅仅提供它定义的属性(类/方法/包/域)的信息。Annotations的用户(同样是一些代码)来读取这些信息并实现必要的逻辑。当我们使用Java的标注Annotations(例如@Override)时，JVM就是一个用户，它在字节码层面工作
    
    J2SE5.0版本在 java.lang.annotation提供了四种元注解，专门注解其他的注解：
    
    - @Documented –注解是否将包含在JavaDoc中，表示是否将注解信息添加在java文档中
    - @Retention –什么时候使用该注解，定义该注解的生命周期。
    - @Target? –注解用于什么地方，如果不明确指出，该注解可以放在任何地方。
    - @Inherited – 是否允许子类继承该注解
   
**RetentionPolicy.SOURCE** 在编译阶段丢弃。这些注解在编译结束之后就不再有任何意义，所以它们不会写入字节码。@Override, @SuppressWarnings都属于这类注解。

**RetentionPolicy.CLASS**  在类加载的时候丢弃。在字节码文件的处理中有用。注解默认使用这种方式。

**RetentionPolicy.RUNTIME**  始终不会丢弃，运行期也保留该注解，因此可以使用反射机制读取该注解的信息。我们自定义的注解通常使用这种方式。

**ElementType.TYPE**:用于描述类、接口或enum声明

**ElementType.FIELD**:用于描述实例变量

ElementType.METHOD

ElementType.PARAMETER

ElementType.CONSTRUCTOR

ElementType.LOCAL_VARIABLE

**ElementType.ANNOTATION_TYPE** 另一个注释

**ElementType.PACKAGE** 用于记录java文件的package信息

###例子###

	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.TYPE)
	public @interface Assignment {
      String assignee();
      int effort();
      double finished() default 0;
	} 

JDK 5中提供了**apt工具用来对注解进行处理**。apt是一个命令行工具，与之配套的还有一套用来描述程序语义结构的Mirror API。Mirror API（com.sun.mirror.*）描述的是程序在编译时刻的静态结构。**通过Mirror API可以获取到被注解的Java类型元素的信息**，从而提供相应的处理逻辑。具体的处理工作交给apt工具来完成。编写注解处理器的核心是AnnotationProcessorFactory和AnnotationProcessor两个接口。后者表示的是注解处理器，而前者则是为某些注解类型创建注解处理器的工厂。

	public class AssignmentApf implements AnnotationProcessorFactory {  
    public AnnotationProcessor getProcessorFor(Set<AnnotationTypeDeclaration> atds,? AnnotationProcessorEnvironment env) {
        if (atds.isEmpty()) {
           return AnnotationProcessors.NO_OP;
        }
        return new AssignmentAp(env); //返回注解处理器
    } 
    public Collection<String> supportedAnnotationTypes() {
        return Collections.unmodifiableList(Arrays.asList("annotation.Assignment"));
    }
    public Collection<String> supportedOptions() {
        return Collections.emptySet();
    }
	}

> AnnotationProcessorFactory接口有三个方法：getProcessorFor是根据注解的类型来返回特定的注解处理器；supportedAnnotationTypes是返回该工厂生成的注解处理器所能支持的注解类型；supportedOptions用来表示所支持的附加选项。在运行apt命令行工具的时候，可以通过-A来传递额外的参数给注解处理器，如-Averbose=true。当工厂通过 supportedOptions方法声明了所能识别的附加选项之后，注解处理器就可以在运行时刻通过AnnotationProcessorEnvironment的getOptions方法获取到选项的实际值。注解处理器本身的基本实现如下所示。

	public class AssignmentAp implements AnnotationProcessor { 
    private AnnotationProcessorEnvironment env;
    private AnnotationTypeDeclaration assignmentDeclaration;
    public AssignmentAp(AnnotationProcessorEnvironment env) {
        this.env = env;
        assignmentDeclaration = (AnnotationTypeDeclaration) env.getTypeDeclaration("annotation.Assignment");
    }
    public void process() {
        Collection<Declaration> declarations = env.getDeclarationsAnnotatedWith(assignmentDeclaration);
        for (Declaration declaration : declarations) {
           processAssignmentAnnotations(declaration);
        }
    }
    private void processAssignmentAnnotations(Declaration declaration) {
        Collection<AnnotationMirror> annotations = declaration.getAnnotationMirrors();
        for (AnnotationMirror mirror : annotations) {
            if (mirror.getAnnotationType().getDeclaration().equals(assignmentDeclaration)) {
                Map<AnnotationTypeElementDeclaration, AnnotationValue> values = mirror.getElementValues();
                String assignee = (String) getAnnotationValue(values, "assignee"); //获取注解的值
            }
        }
    }  

> 注解处理器的处理逻辑都在process方法中完成。通过一个声明（Declaration）的getAnnotationMirrors方法就可以获取到该声明上所添加的注解的实际值。得到这些值之后，处理起来就不难了。

上面介绍的做法都是在编译时刻进行处理的。而有些时候则需要在运行时刻来完成对注解的处理。这个时候就需要用到Java的反射API。反射API提供了在运行时刻读取注解信息的支持。不过前提是注解的保留策略声明的是运行时。Java反射API的AnnotatedElement接口提供了获取类、方法和域上的注解的实用方法。比如获取到一个Class类对象之后，通过getAnnotation方法就可以获取到该类上添加的指定注解类型的注解。

###实例分析###

下面通过一个具体的实例来分析说明在实践中如何来使用和处理注解。假定有一个公司的雇员信息系统，从访问控制的角度出发，对雇员的工资的更新只能由具有特定角色的用户才能完成。考虑到访问控制需求的普遍性，可以定义一个注解来让开发人员方便的在代码中声明访问控制权限。

	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.METHOD)
	public @interface RequiredRoles {
    String[] value();
	}

下一步则是如何对注解进行处理，这里使用的Java的反射API并结合动态代理。下面是动态代理中的InvocationHandler接口的实现。

	public class AccessInvocationHandler<T> implements InvocationHandler {
    final T accessObj;
    public AccessInvocationHandler(T accessObj) {
        this.accessObj = accessObj;
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        RequiredRoles annotation = method.getAnnotation(RequiredRoles.class); //通过反射API获取注解
        if (annotation != null) {
            String[] roles = annotation.value();
            String role = AccessControl.getCurrentRole();
            if (!Arrays.asList(roles).contains(role)) {
                throw new AccessControlException("The user is not allowed to invoke this method.");
            }
        }
        return method.invoke(accessObj, args);
    } 
	} 
> 在具体使用的时候，首先要通过Proxy.newProxyInstance方法创建一个EmployeeGateway的接口的代理类，使用该代理类来完成实际的操作
