---
layout: post
title: "使用Eclipse AST来解析Java"
date: 2013-06-19 14:46
comments: true
categories: java
tags: [ eclipse, ast ]
---
Eclipse中有个Outline视图，这个视图内可以显示类的各种信息，包括属性、方法等。同时在这个视图内点击哪里，就能定位的类什么地方。这是个神奇的东西，那么它是如何实现的呢？其实就是AST（Abstract Syntax Tree），抽象语法树了。利用这个来对Java源码进行解析。
另外，Eclipse有一个[AST View](http://www.eclipse.org/jdt/ui/astview/)插件，使用这个能更加的清楚的看到解析后的效果。

![Eclipse AST View](/images/blog/eclipse-ast-view.png)

<!--more-->
#### 依赖的jar包

- org.eclipse.core.contenttype_3.4.100.v20110423-0524.jar
- org.eclipse.core.jobs_3.5.101.v20120113-1953.jar
- org.eclipse.core.resources_3.7.101.v20120125-1505.jar
- org.eclipse.core.runtime_3.7.0.v20110110.jar
- org.eclipse.equinox.common_3.6.0.v20110523.jar
- org.eclipse.equinox.preferences_3.4.2.v20120111-2020.jar
- org.eclipse.jdt.core_3.7.3.v20120119-1537.jar
- org.eclipse.osgi_3.7.2.v20120110-1415.jar

这些包都可以在eclipse的plugin目录找到。可能版本有所区别。
如果使用maven管理，经个人试验，直接使用以下dependency也是可以的。

              <dependency>
                    <groupId>org.eclipse.tycho</groupId>
                    <artifactId>org.eclipse.jdt.core</artifactId>
                    <version>3.8.0.v_C03</version>
                </dependency>
                 <dependency>
                     <groupId>org.eclipse.core</groupId>
                     <artifactId>org.eclipse.core.runtime</artifactId>
                     <version>3.6.0.v20100505</version>
                 </dependency>
                <dependency>
                    <groupId>org.eclipse.core</groupId>
                    <artifactId>org.eclipse.core.resources</artifactId>
                    <version>3.6.0.v20100526-0737</version>
                </dependency>

#### 如何解析Java

    package cn.shenyanchao.ast;

    import org.apache.commons.io.FileUtils;
    import org.eclipse.jdt.core.dom.*;

    import java.io.File;
    import java.io.IOException;
    import java.util.List;

    /**
     * Created with IntelliJ IDEA.
     *
     * @author shenyanchao
     *         Date:  6/13/13
     *         Time:  11:52 AM
     */
    public class AstAnalyzer {
        public static void main(String[] args) throws IOException {

            String javaSource = FileUtils.readFileToString(new File("/home/shenyanchao/IdeaProjects/ast/src/main/java/cn/shenyanchao/from/ShenYanChaoAST.java"));

            ASTParser parser = ASTParser.newParser(AST.JLS3);
            parser.setSource(javaSource.toCharArray());

            // 使用解析器进行解析并返回AST上下文结果(CompilationUnit为根节点)
            CompilationUnit result = (CompilationUnit) parser.createAST(null);

            result.imports();
            result.getPackage();
            result.getCommentList();
            System.out.println(result.getCommentList().toString());

            TypeDeclaration type = (TypeDeclaration) result.types().get(0);
            System.out.println("---------Type---------");
            System.out.println(type.toString());

            MethodDeclaration method = type.getMethods()[0];
            method.parameters();
            method.isConstructor();

            System.out.println("---------Method---------");
            System.out.println(method.toString());
            method.getName();
            method.getModifiers();
            Type returnType = method.getReturnType2();
            System.out.println("returnType = " + returnType.toString());


            Block methodBody = method.getBody();
            List<Statement> statementList = methodBody.statements();

            System.out.println(statementList.toString());

            statementList.get(0);

            ExpressionStatement ifs = (ExpressionStatement) method.getBody().statements().get(1);
            Assignment expression = (Assignment) ifs.getExpression();
            Expression exp = expression.getRightHandSide();

            System.out.println(result.toString());
        }
    }

#### 如何创建Java

    package cn.shenyanchao.ast;

    import org.eclipse.jdt.core.dom.*;

    /**
     * Created with IntelliJ IDEA.
     *
     * @author shenyanchao
     *         Date:  6/13/13
     *         Time:  11:27 AM
     */
    public class AstHelloWorld {


        public static void main(String[] args) {
            AST ast = AST.newAST(AST.JLS3);
            CompilationUnit compilationUnit = ast.newCompilationUnit();

            // 创建类
            TypeDeclaration programClass = ast.newTypeDeclaration();
            programClass.setName(ast.newSimpleName("HelloWorld"));
            programClass.modifiers().add(
                    ast.newModifier(Modifier.ModifierKeyword.PUBLIC_KEYWORD));
            compilationUnit.types().add(programClass);

            // 创建包
            PackageDeclaration packageDeclaration = ast.newPackageDeclaration();
            packageDeclaration.setName(ast.newName("cn.shenyanchao.hello"));
            compilationUnit.setPackage(packageDeclaration);

            MethodDeclaration main = ast.newMethodDeclaration();
            main.setName(ast.newSimpleName("main"));
            main.modifiers().add(
                    ast.newModifier(Modifier.ModifierKeyword.PUBLIC_KEYWORD));
            main.modifiers().add(ast.newModifier(Modifier.ModifierKeyword.STATIC_KEYWORD));
            main.setReturnType2(ast.newPrimitiveType(PrimitiveType.VOID));
            programClass.bodyDeclarations().add(main);
            Block mainBlock = ast.newBlock();
            main.setBody(mainBlock);

            // 给main方法定义String[]参数
            SingleVariableDeclaration mainParameter = ast
                    .newSingleVariableDeclaration();
            mainParameter.setName(ast.newSimpleName("arg"));
            mainParameter.setType(ast.newArrayType(ast.newSimpleType(ast
                    .newName("String"))));
            main.parameters().add(mainParameter);

            MethodInvocation println = ast.newMethodInvocation();
            println.setName(ast.newSimpleName("println"));

            //生成String类型的常量
            StringLiteral s = ast.newStringLiteral();
            s.setLiteralValue("Hello World");
            println.arguments().add(s);

            println.setExpression(ast.newName("System.out"));

            mainBlock.statements().add(ast.newExpressionStatement(println));

            System.out.println(compilationUnit.toString());
        }
    }

具体的使用手册，参见<http://www.shenyanchao.cn/blog/2013/06/07/eclipse-ast/>

