### 1. 引言

如下代码所示：

```java
public void processColor {
	if("Red".equalsIgnoreCase(color)) {
    processRed();
  } else if("Yellow".equalsIgnoreCase(color)) {
    processYellow();
  } else if("Black".equalsIgnoreCase(color)) {
    processBlack();
  } else {
    //Do nothing
  }
}
```

在某些情况下，我们需要像上面那样做多个 if-else 条件。有人可能更喜欢使用[switch case](https://www.java67.com/2022/02/java-switch-statement-and-expression.html)，但它不会提供[干净和健壮的代码](https://medium.com/javarevisited/7-best-courses-to-learn-refactoring-and-clean-coding-in-java-47bea3c67006)。

那么我们如何才能删除这些多个 if-else 条件呢？

### 2. 解决方案

我们可以用地图来做到这一点。

- 对于String字符串类型，您可以使用HashMap
- 如果您有Enum，请使用EnumMap而不是 HashMap。

```java
public class ColorProcessor {
  public void processColor {
  Map<String, Supplier<String>> colorProcessorMap;
  
  ColorProcessor() {
    colorProcessorMap.put("red",this::processRed());
    colorProcessorMap.put("yellow",this::processYellow());
    colorProcessorMap.put("black",this::processBlack());
  }
    
	public String processColor(String color) {
    if(colorProcessorMap.containsKey(color)) {
      return colorProcessorMap.get(color).get();
    } else {
      return "Invalid color"
    }
  }
    
  String processRed() { return "Red is processed";}
    
  String processYellow() { return "Yellow is processed";}
    
  String processBlack() { return "Black is processed";}
}

```

我们不使用 if-else 条件，而是使用Map解决

- 确保您的条件符合Map。
- 我们没有为每个案例编写逻辑处理，而是有一个映射，我们将案例和逻辑作为*键值*对。
- 因此，我们可以根据键从映射中检索逻辑。
