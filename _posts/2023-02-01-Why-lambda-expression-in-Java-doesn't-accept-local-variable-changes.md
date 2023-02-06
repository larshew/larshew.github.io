---
layout: post
title:  "Why lambda expression in Java doesn't accept modifying a local variable?"
date:   2023-02-01 16:29:31 +0900
categories: java
---
<h1>Why lambda expression in Java doesn't accept modifying a local variable?</h1>

Currently, I am studying Java concurrency and I have noticed that this code works.<br>
```java
class Example{
    static int a=0;
    public static void main(String[] args){
        Runnable runnable = () -> {
            for (int i=1;i<=1_000_000;i++){
                a++; // why can I modify outer value in lambda expression?
            }
        };

        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.run();
        thread2.run();
        try{
            thread1.join();
            thread2.join();
        } catch(InterruptedException e){
            // Handles InterruptedException.
        }
        System.out.println(a); // a is expected to be 2_000_000, but isn't.
    }
}
```
I didn't know how lambda expression exactly works and just thought it is impossible to modify external variable(including fields). But that was my misconception.<br><br>
Generally, lambda expression doesn't have to restrict modifying variables, but if Java permits to modify <b>local variables</b> like this, there could be trouble.
```java
class Example2{
    public static Runnable foo(){
        int x=1;
        return ()->{x++;};
    }
    public static void main(String[] args){
        Runnable runnable = foo();
        runnable.run(); // x would be x++, 
        // but does variable "x" exist?
    }
}
```
The function `foo()` executed and `foo()` returns lambda function which executes `x = x+1`. and this lambda function is executed in main function. <br>
However, in main function, is `x` valid? Maybe not, because x is only valid when `foo()` is running. Therefore, Java restricts modifying external local variables, and just enables to use "final" or "effectively final" variables, which can be <b>captured</b> when executing lambda expression.
