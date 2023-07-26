# 14장. 점진적 개선

- 이 장은 지속적으로 코드를 추가하고 개선하는 내용이 이어짐. 
- 어떻게 코드를 지속적으로 고쳐나가는지, 자신의 코딩 스타일과 비교하며 보면 재미있는 내용인 것 같습니다.
- (왜 무려 50장이나 되는거지.... 책에서 diff 만 좀 올려주지.... 휴.. ) 

## String 인수

### String 인수의 추가

```java
// p. 272 ~ 273
// string 을 추가하는 과정입니다.
// stringArgs 와 getter / setter 가 추가되었습니다. 

// 1. stringArgs 추가.
// Map 객체 stringArgs 로 ArgumentMarshaler 를 value 로 가지는 변수를 만듭니다. 
private Map < Character, ArgumentMarshaler > stringArgs =
    new HashMap < Character, ArgumentMarshaler > ();
// 
// 2. parse 된 경우에 stringArgs 에 StringArgumentMarshaler 새 객체를 넣습니다. 
private void parseStringSchemaElement(char elementId) {
    stringArgs.put(elementId, new StringArgumentMarshaler());
}
// 
// 3. setStringArg 가 호출 될 경우에 Map인 stringArgs 에서 key (argChar) 로 ArgumentMarshaler 를 찾아 거기에 setString 을 합니다. 
private void setStringArg(char argChar) throws ArgsException {
    currentArgument++;
    try {
        stringArgs.get(argChar).setString(args[currentArgument]);    // 여기!! 
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_STRING;
        throw new ArgsException();
    }
}
//
// 4. getString 의 경우 stringArgs 에서 arg로 get 하여 ArgumentMarshaler 를 새 객체 am으로 생성하여 반환하여 전달합니다. 
public String getString(char arg) {
    Args.ArgumentMarshaler am = stringArgs.get(arg);
    return am == null ? "" : am.getString();
}
//
// 5. ArgumentMarshaler 내부의 getter / setter 정의. 
private class ArgumentMarshaler {
    private boolean booleanValue = false;
    private String stringValue;
    public void setBoolean(boolean value) {
        booleanValue = value;
    }
    public boolean getBoolean() {
        return booleanValue;
    }

    // getter & setter 를 새로 정의 합니다. 
    public void setString(String s) {
        stringValue = s;
    }
    public String getString() {
        return stringValue == null ? "" : stringValue;
    }
}
```


```java
// p. 274 ~ 275
// int 를 추가합니다. 
// 앞선 String 과 유사합니다. int 에 대한 예외 정도가 좀 더 추가되는 것을 볼 수 있습니다. 

// 1. Map 객체인 intArgs 를 추가 합니다.
private Map < Character, ArgumentMarshaler > intArgs =
    new HashMap < Character, ArgumentMarshaler > ();
//
// 2. Int 를 parse 할 때 put 할 parseIntergerSchemaElement 를 정의 합니다. 
private void parseIntegerSchemaElement(char elementId) {
    intArgs.put(elementId, new IntegerArgumentMarshaler());
}
//
// 3. intArgs 에 값을 넣는 setIntArg 를 만듭니다.
private void setIntArg(char argChar) throws ArgsException {
    currentArgument++;
    String parameter = null;
    try {
        parameter = args[currentArgument];

        // map 객체인 intArgs 에 key 로 찾아 Argumentmarshaler 에 setInterger 합니다. 
        intArgs.get(argChar).setInteger(Integer.parseInt(parameter));
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_INTEGER;
        throw new ArgsException();
    } catch (NumberFormatException e) {
        valid = false;
        errorArgumentId = argChar;
        errorParameter = parameter;
        errorCode = ErrorCode.INVALID_INTEGER;
        throw new ArgsException();
    }
}
//
// 4. getInt 할경우 Argumentmarshaler 를새로 생성하여 값을 담아서 반환 합니다. 
public int getInt(char arg) {
    Args.ArgumentMarshaler am = intArgs.get(arg);
    return am == null ? 0 : am.getInteger();
}
//
// 5. getter / setter 생성. 
private class ArgumentMarshaler {
    private boolean booleanValue = false;
    private String stringValue;
    private int integerValue;
    public void setBoolean(boolean value) {
        booleanValue = value;
    }
    public boolean getBoolean() {
        return booleanValue;
    }
    public void setString(String s) {
        stringValue = s;
    }
    public String getString() {
        return stringValue == null ? "" : stringValue;
    }

    // setter / getter 생성
    public void setInteger(int i) {
        integerValue = i;
    }
    public int getInteger() {
        return integerValue;
    }
}
```


```java
// p. 275 ~ 276
// 모든 논리 (Boolean, string, int ?) 가 다 준비 되었으니 파생 클래서를 만들어 기능 분산을 합니다.
// boolean 을 먼저 분리하여 BooleanArgumentMarshaler 로 만들어 호출되는지 확인합니다.
// 첫번째 단계로 추상함수 set 을 만듭니다. 

private abstract class ArgumentMarshaler {
    protected boolean booleanValue = false;    // 1. 여기가 변경. private -> protected
    private String stringValue;
    private int integerValue;
    public void setBoolean(boolean value) {
        booleanValue = value;
    }
    public boolean getBoolean() {
        return booleanValue;
    }
    public void setString(String s) {
        stringValue = s;
    }
    public String getString() {
        return stringValue == null ? "" : stringValue;
    }
    public void setInteger(int i) {
        integerValue = i;
    }
    public int getInteger() {
        return integerValue;
    }
    public abstract void set(String s);   // 2. 추상함수 set 을 만듭니다. 
}
```


```java
// p. 276
// 다음 BooleanargumentMarshaler 에 set 을 구현합니다.
// 사실 아래 String s 를 받아서 하는 일은 아무것도 없습니다.
// 하지만, 다른 함수를 만들때 set 에서 string 을 받아 처리해야 하는 부분이 있기 때문에
// 형태를 같게 맞추어 주기 위하여 String을 받는 형태로 합니다. 

private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    public void set(String s) {
        booleanValue = true;
    }
}

```


```java
// p. 276
// setBoolean 호출을 set 으로 바꿉니다.

private void setBooleanArg(char argChar, boolean value) {
//    booleanArgs.get(argChar).setBoolean(value);     // 이전엔 이랬습니다. 
    booleanArgs.get(argChar).set("true");
}

```
이 상태로 테스트를 통과할 수 있고, 

이제 set을 BooleanArgumentMarshaler 로 옮겼으므로 

ArgumentMarshaler 에서 setBoolean 을 제거합니다.

```java
private abstract class ArgumentMarshaler {
    protected boolean booleanValue = false;
    private String stringValue;
    private int integerValue;
    //public void setBoolean(boolean value) {    // 여기를 없애주는겁니다. 
    //    booleanValue = value;
    //}
// 중략
}
```



```java
// p. 277
// get 을 BooleanArgumentMarshaler 로 옮깁니다. 

// 원래 형태는 다음의 형태입니다. 
public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = booleanArgs.get(arg);
    return am.getBoolean();
}

// 아래와 같이 옮겨집니다. 
public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = booleanArgs.get(arg);
    return am != null && (Boolean) am.get();  // 일단 get 으로 대충 가져온 값을 boolean 을 내야 하니 casting 이 추가됩니다. 
}
```


```java
// p. 277
// ArgumentMarshaler 에서 공통으로 부를 get 이 필요합니다.
// 우선 Object 로 (대충 어떤거든) return 하는 형태를 가지도록 만듭니다. 

private abstract class ArgumentMarshaler {
    //

    public Object get() {
        return null;
    }
}

```

- 컴파일: pass 됩니다. 
- 테스트: fail 됩니다. 

(이유?)
- BooleanArgumentMarshaler 에서 get 을 해야 하는데 하지 않았고 (함수조차 만들지 않았음. ) 
- ArgumentMarshaler 에서 만든 get 에서 null 을 반환하는데, 이를 casting 합니다. (의도한 결과 아님) 

```java
// p. 277 ~ 278
// 고치기 위해, ArgumentMarshaler 에서는 get 을 abstract 로 하고, BooleanArgumentMarshaler 에서 이를 구현합니다. 
private abstract class ArgumentMarshaler {
    protected boolean booleanValue = false;
    //

    public abstract Object get();    // 1. abstract !!!
}
private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    public void set(String s) {
        booleanValue = true;
    }
    public Object get() {              // 2. 구현!! 
        return booleanValue;
    }
}

```

여기까지 하면 
- Boolean 에 대하여 모든 것이 옮겨지고,
- 컴파일, 테스트 pass. 

- ArgumentMarshaler 에서 사용하지 않는 getBoolean 함수 제거
- protected 변수인 booleanValue 를 BooleanArgumentMarshaler 로 내려 private 변수로 선언.

이후 같은 방법으로 String / int 에 대해서도 동일하게 진행 합니다. 

(String / int 에 대해서는 생략 합니다. )

int 의 경우 NumberFormatException 같은 숫자에 대한 exception 도 IntArgumentMarshaler 로 넣으면, 

ArgumentMarshaler 에서 구체적인 각 형태에 대한 예외를 하지 않게 되어 더 깔끔해집니다. 


이후 booleanArgs, stringArgs, intArgs 의 세개의 Map 을 정리하도록 접근 해봅니다. 

```java
// p. 281 ~ 282
// marshalers 추가. booleanArgs, stringArgs, intArgs 대신 연결하기

public class Args {
    // 
    private Map < Character, ArgumentMarshaler > booleanArgs =
        new HashMap < Character, ArgumentMarshaler > ();
    private Map < Character, ArgumentMarshaler > stringArgs =
        new HashMap < Character, ArgumentMarshaler > ();
    private Map < Character, ArgumentMarshaler > intArgs =
        new HashMap < Character, ArgumentMarshaler > ();
    private Map < Character, ArgumentMarshaler > marshalers =  // marshalers 를 새로 만들고 
        new HashMap < Character, ArgumentMarshaler > ();
    // 
    private void parseBooleanSchemaElement(char elementId) {
        ArgumentMarshaler m = new BooleanArgumentMarshaler();
        booleanArgs.put(elementId, m);
        marshalers.put(elementId, m);                             // marshalers 로 연결해줍니다. 
    }
    private void parseIntegerSchemaElement(char elementId) {
        ArgumentMarshaler m = new IntegerArgumentMarshaler();
        intArgs.put(elementId, m);
        marshalers.put(elementId, m);                             // marshalers 로 연결해줍니다. 
    }
    private void parseStringSchemaElement(char elementId) {
        ArgumentMarshaler m = new StringArgumentMarshaler();
        stringArgs.put(elementId, m);
        marshalers.put(elementId, m);                             // marshalers 로 연결해줍니다. 
    }
}
```

```java
// p. 282
// 참고. A instanceof B : A 가 B 클래스 타입인지 확인하는 함수

private boolean isBooleanArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof BooleanArgumentMarshaler;
}
private boolean isIntArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof IntegerArgumentMarshaler;
}
private boolean isStringArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof StringArgumentMarshaler;
}

```

```java
// p. 283
// marshalers.get 을 호출하는 코드로 정리.

private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);                // 여기
    if (isBooleanArg(m))
        setBooleanArg(argChar);
    else if (isStringArg(m))
        setStringArg(argChar);
    else if (isIntArg(m))
        setIntArg(argChar);
    else
        return false;
    return true;
}
private boolean isIntArg(ArgumentMarshaler m) {                      // 여기
    return m instanceof IntegerArgumentMarshaler;
}
private boolean isStringArg(ArgumentMarshaler m) {                    // 여기
    return m instanceof StringArgumentMarshaler;
}
private boolean isBooleanArg(ArgumentMarshaler m) {                    // 여기
    return m instanceof BooleanArgumentMarshaler;
}

```

```java
// p. 283
// 위의 코드 중 굳이 함수를 부르지 않아도 됨....;;
// isxxxArg 등이 빈번히 호출딜 일이 없으니.
// ->코드 변경

// 이전 코드
    if (isBooleanArg(m))  // isBooleanArg 는 여기 한곳에만 있어요. 
        setBooleanArg(argChar);
    else if (isStringArg(m))  // isStringArg 는 여기 한곳에만 있어요. 
        setStringArg(argChar);
    else if (isIntArg(m))  // isIntArg 는 여기 한곳에만 있어요. 
        setIntArg(argChar);
    else
        return false;
//

// 바로 위에서 작성한 이 코드를 넣어요.
private boolean isIntArg(ArgumentMarshaler m) {
    return m instanceof IntegerArgumentMarshaler;  // 이 코드를 바로 넣어요. 
}
private boolean isStringArg(ArgumentMarshaler m) {
    return m instanceof StringArgumentMarshaler;  // 이 코드를 바로 넣어요. 
}
private boolean isBooleanArg(ArgumentMarshaler m) {
    return m instanceof BooleanArgumentMarshaler;  // 이 코드를 바로 넣어요. 
}
// 


private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m instanceof BooleanArgumentMarshaler)      // 인라인 함수로 대신해요. 
        setBooleanArg(argChar);
    else if (m instanceof StringArgumentMarshaler)  // 인라인 함수로 대신해요. 
        setStringArg(argChar);
    else if (m instanceof IntegerArgumentMarshaler)  // 인라인 함수로 대신해요. 
        setIntArg(argChar);
    else
        return false;
    return true;
}

```
(다들 아시는 내용일수도 있지만!!! )

여러개의 공통된 코드가 있다면, 당연히 함수화 하는것이 코드 중복을 막아서 좋지만,

불필요한 함수를 굳이 사용할 필요는 없습니다.

<details>
<summary>불필요한 함수를 call 하게 되면.. </summary>

함수가 호출되면 사용중인 변수 등의 환경을 모두 stack 에 집어넣고, 

새로 시작되는 부분에 대한 base stack 을 다시 잡고 (sp(stack pointer) 변경), 

새로 호출되는 부분으로 pc(program counter)를 조절하고,  

......  

등을 조절하는 과정을 거치게 되는데, 

굳이 보기 좋자고 이런걸 할 필요는 없을꺼 같아요... 

1. 참고 - Risc (ARM processor) 기준 register 구조.

![image](https://github.com/yoons6548/clean_code/assets/45020799/2007499b-5d3f-46d6-b7a7-b10ebe5e0f1e)

(출처 : https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/how-to-call-a-function-from-arm-assembler )

2. Cisc (IA32 : Inter Architecture 32bit) 기준 register 구조.

![image](https://github.com/yoons6548/clean_code/assets/45020799/a7f8709a-303d-4e5a-a967-47b86ee02fdc)

(출처 : https://www.cs.cmu.edu/afs/cs/academic/class/15740-f18/www/lectures/02-isa.pdf )
</details>





```java
// p. 284
// set 함수에서 기존 HashMap 을 marshalers HashMap 으로 변경합니다.
// Boolean 부터 먼저 시작 합니다. 

private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m instanceof BooleanArgumentMarshaler)
        setBooleanArg(m);                            // was: setBooleanArg(argChar);
    else if (m instanceof StringArgumentMarshaler)
        setStringArg(argChar);
    else if (m instanceof IntegerArgumentMarshaler)
        setIntArg(argChar);
    else
        return false;
    return true;
}
//
private void setBooleanArg(ArgumentMarshaler m) {    // was: private void setBooleanArg(char argChar, boolean value) {
    try {
        m.set("true");                               // was: booleanArgs.get(argChar).set("true");
    } catch (ArgsException e) {}
}


```


```java
// p. 284 ~ 285
// set 함수에서 기존 HashMap 을 marshalers HashMap 으로 변경합니다.
// string, int 도 똑같이 진행해줍니다. 

private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    try {
        if (m instanceof BooleanArgumentMarshaler)
            setBooleanArg(m);
        else if (m instanceof StringArgumentMarshaler)
            setStringArg(m);                            // was: setStringArg(argChar);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(m);                            // was: setIntArg(argChar);
        else
            return false;
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true;
}
// was: private void setIntArg(char argChar) throws ArgsException {  
private void setIntArg(ArgumentMarshaler m) throws ArgsException {  -> ArgumentMarshaler 를 받도록 파라메터가 변경 되었습니다. 
    currentArgument++;
    String parameter = null;
    try {
        parameter = args[currentArgument];
        m.set(parameter);                                    // was: intArgs.get(argChar).setInteger(Integer.parseInt(parameter));
    } catch (ArrayIndexOutOfBoundsException e) {
        errorCode = ErrorCode.MISSING_INTEGER;
        throw new ArgsException();
    } catch (ArgsException e) {
        errorParameter = parameter;
        errorCode = ErrorCode.INVALID_INTEGER;
        throw e;
    }
}
// was: private void setStringArg(char argChar) throws ArgsException {
private void setStringArg(ArgumentMarshaler m) throws ArgsException {    // ArgumentMarshaler 를 받도록 파라메터가 변경 되었습니다. 
    currentArgument++;
    try {
        m.set(args[currentArgument]);                          // was: stringArgs.get(argChar).setString(args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
        errorCode = ErrorCode.MISSING_STRING;
        throw new ArgsException();
    }
}

```

이제 map 변수 삼형제(booleanArgs, stringArgs, intArgs)를 제거해 봅니다. 

(marshalers 로 모두 연결 해가니..) 

```java
// p. 285 ~ 286
// Boolean 부터 제거해 봅니다.

// 변경 전
// booleanArgs map 에서 ArgumentMarshaler 를 가져와서 여기에서 get을 합니다.
public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = booleanArgs.get(arg);
    return am != null && (Boolean) am.get();
}

// 변경 후
// 공통으로 사용되는 marshalers 에서 get 을 합니다.
// 혹시, 
public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg);
    boolean b = false;
    try {
        b = am != null && (Boolean) am.get();
    } catch (ClassCastException e) {
        b = false;
    }
    return b;
}
```



```java
// p. 286
// 사용중인 boolean map 을 제거합니다.
private void parseBooleanSchemaElement(char elementId) {
    ArgumentMarshaler m = new BooleanArgumentMarshaler();
    // booleanArgs.put(elementId, m);                      // 제거! 
    marshalers.put(elementId, m);
}

public class Args {
    // 
//    private Map < Character, ArgumentMarshaler > booleanArgs =   // 제거! 
//        new HashMap < Character, ArgumentMarshaler > ();
    private Map < Character, ArgumentMarshaler > stringArgs =
        new HashMap < Character, ArgumentMarshaler > ();
    private Map < Character, ArgumentMarshaler > intArgs =
        new HashMap < Character, ArgumentMarshaler > ();
    private Map < Character, ArgumentMarshaler > marshalers =
        new HashMap < Character, ArgumentMarshaler > ();
    // 
}
```

```java
// p. 287
// 동일하게 string, integer 도 변형하고, 이전 map 제거합니다. 
private void parseBooleanSchemaElement(char elementId) {
  // ArgumentMarshaler m = new BooleanArgumentMarshaler();  // was. 변수(m) 를 생성하여 사용하는 것이 없어졌기 때문에 
  // marshalers.put(elementId, m);                          // 이 자리의 m 자리에서 바로 new Boolean... 을 넣어주는 것으로 바꿉니다. 
  marshalers.put(elementId, new BooleanArgumentMarshaler());
}

private void parseIntegerSchemaElement(char elementId) {

  // ArgumentMarshaler m = new IntegerArgumentMarshaler();  // was. 위와 동일한 방식으로 바꿉니다. (m 대신 new Integer.. ) 
  // intArgs.put(elementId, m);
  // marshalers.put(elementId, m);                             
  marshalers.put(elementId, new IntegerArgumentMarshaler());
}
private void parseStringSchemaElement(char elementId) {
  marshalers.put(elementId, new StringArgumentMarshaler());
}
// 
public String getString(char arg) {
  // Args.ArgumentMarshaler am = stringArgs.get(arg);  // was. StringArgs 대신 marshalers 로 교체
  Args.ArgumentMarshaler am = marshalers.get(arg);
  try {
    return am == null ? "" : (String) am.get();
  } catch (ClassCastException e) {                      // am 을 String 으로 cast 하는 것에 대한 예외처리가 추가 됩니다. 
    return "";
  }
}
public int getInt(char arg) {
  // Args.ArgumentMarshaler am = intArgs.get(arg);   // was. intArgs 대신 marshalers 로 교체
  Args.ArgumentMarshaler am = marshalers.get(arg);
  try {
    return am == null ? 0 : (Integer) am.get();
  } catch (Exception e) {                             // exception 처리. 실패시 0 으로 반환하도록 합니다. 
    return 0;
  }
}

// 
public class Args {
  // 
  // private Map < Character, ArgumentMarshaler > stringArgs =  // 삭제
  //   new HashMap < Character, ArgumentMarshaler > ();         // 삭제
  // private Map < Character, ArgumentMarshaler > intArgs =     // 삭제
  //   new HashMap < Character, ArgumentMarshaler > ();         // 삭제
  private Map < Character, ArgumentMarshaler > marshalers =
    new HashMap < Character, ArgumentMarshaler > ();
}

```


```java
// p. 288
// parse 함수 세개를 인라인 함수로 바꿉니다. 
private void parseSchemaElement(String element) throws ParseException {
  char elementId = element.charAt(0);
  String elementTail = element.substring(1);
  validateSchemaElementId(elementId);
  if (isBooleanSchemaElement(elementTail))
    marshalers.put(elementId, new BooleanArgumentMarshaler());    // 여기
  else if (isStringSchemaElement(elementTail))
    marshalers.put(elementId, new StringArgumentMarshaler());     // 여기
  else if (isIntegerSchemaElement(elementTail)) {
    marshalers.put(elementId, new IntegerArgumentMarshaler());    // 여기
  } else {
    throw new ParseException(String.format(
      "Argument: %c has invalid format: %s.", elementId, elementTail), 0);
  }
}

```

<details>
<summary>그래서 다 하고나면 지금까지 이런 형태가 됩니다. </summary>

```java
// p. 288 ~ 295
package com.objectmentor.utilities.getopts;
import java.text.ParseException;
import java.util.*;
public class Args {
  private String schema;
  private String[] args;
  private boolean valid = true;
  private Set < Character > unexpectedArguments = new TreeSet < Character > ();
  private Map < Character, ArgumentMarshaler > marshalers =
    new HashMap < Character, ArgumentMarshaler > ();
  private Set < Character > argsFound = new HashSet < Character > ();
  private int currentArgument;
  private char errorArgumentId = '\0';
  private String errorParameter = "TILT";
  private ErrorCode errorCode = ErrorCode.OK;
  private enum ErrorCode {
    OK,
    MISSING_STRING,
    MISSING_INTEGER,
    INVALID_INTEGER,
    UNEXPECTED_ARGUMENT
  }
  public Args(String schema, String[] args) throws ParseException {
    this.schema = schema;
    this.args = args;
    valid = parse();
  }
  private boolean parse() throws ParseException {
    if (schema.length() == 0 && args.length == 0)
      return true;
    parseSchema();
    try {
      parseArguments();
    } catch (ArgsException e) {}
    return valid;
  }
  private boolean parseSchema() throws ParseException {
    for (String element: schema.split(",")) {
      if (element.length() > 0) {
        String trimmedElement = element.trim();
        parseSchemaElement(trimmedElement);
      }
    }
    return true;
  }
  private void parseSchemaElement(String element) throws ParseException {
    char elementId = element.charAt(0);
    String elementTail = element.substring(1);
    validateSchemaElementId(elementId);
    if (isBooleanSchemaElement(elementTail))
      marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (isStringSchemaElement(elementTail))
      marshalers.put(elementId, new StringArgumentMarshaler());
    else if (isIntegerSchemaElement(elementTail)) {
      marshalers.put(elementId, new IntegerArgumentMarshaler());
    } else {
      throw new ParseException(String.format(
        "Argument: %c has invalid format: %s.", elementId, elementTail), 0);
    }
  }
  private void validateSchemaElementId(char elementId) throws ParseException {
    if (!Character.isLetter(elementId)) {
      throw new ParseException(
        "Bad character:" + elementId + "in Args format: " + schema, 0);
    }
  }
  private boolean isStringSchemaElement(String elementTail) {
    return elementTail.equals("*");
  }
  private boolean isBooleanSchemaElement(String elementTail) {
    return elementTail.length() == 0;
  }
  private boolean isIntegerSchemaElement(String elementTail) {
    return elementTail.equals("#");
  }
  private boolean parseArguments() throws ArgsException {
    for (currentArgument = 0; currentArgument < args.length; currentArgument++) {
      String arg = args[currentArgument];
      parseArgument(arg);
    }
    return true;
  }
  private void parseArgument(String arg) throws ArgsException {
    if (arg.startsWith("-"))
      parseElements(arg);
  }
  private void parseElements(String arg) throws ArgsException {
    for (int i = 1; i < arg.length(); i++)
      parseElement(arg.charAt(i));
  }
  private void parseElement(char argChar) throws ArgsException {
    if (setArgument(argChar))
      argsFound.add(argChar);
    else {
      unexpectedArguments.add(argChar);
      errorCode = ErrorCode.UNEXPECTED_ARGUMENT;
      valid = false;
    }
  }
  private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    try {
      if (m instanceof BooleanArgumentMarshaler)
        setBooleanArg(m);
      else if (m instanceof StringArgumentMarshaler)
        setStringArg(m);
      else if (m instanceof IntegerArgumentMarshaler)
        setIntArg(m);
      else
        return false;
    } catch (ArgsException e) {
      valid = false;
      errorArgumentId = argChar;
      throw e;
    }
    return true;
  }
  private void setIntArg(ArgumentMarshaler m) throws ArgsException {
    currentArgument++;
    String parameter = null;
    try {
      parameter = args[currentArgument];
      m.set(parameter);
    } catch (ArrayIndexOutOfBoundsException e) {
      errorCode = ErrorCode.MISSING_INTEGER;
      throw new ArgsException();
    } catch (ArgsException e) {
      errorParameter = parameter;
      errorCode = ErrorCode.INVALID_INTEGER;
      throw e;
    }
  }
  private void setStringArg(ArgumentMarshaler m) throws ArgsException {
    currentArgument++;
    try {
      m.set(args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
      errorCode = ErrorCode.MISSING_STRING;
      throw new ArgsException();
    }
  }
  private void setBooleanArg(ArgumentMarshaler m) {
    try {
      m.set("true");
    } catch (ArgsException e) {}
  }
  public int cardinality() {
    return argsFound.size();
  }
  public String usage() {
    if (schema.length() > 0)
      return "-[" + schema + "]";
    else
      return "";
  }
  public String errorMessage() throws Exception {
    switch (errorCode) {
    case OK:
      throw new Exception("TILT: Should not get here.");
    case UNEXPECTED_ARGUMENT:
      return unexpectedArgumentMessage();
    case MISSING_STRING:
      return String.format("Could not find string parameter for -%c.",
        errorArgumentId);
    case INVALID_INTEGER:
      return String.format("Argument -%c expects an integer but was '%s'.",
        errorArgumentId, errorParameter);
    case MISSING_INTEGER:
      return String.format("Could not find integer parameter for -%c.",
        errorArgumentId);
    }
    return "";
  }
  private String unexpectedArgumentMessage() {
    StringBuffer message = new StringBuffer("Argument(s) -");
    for (char c: unexpectedArguments) {
      message.append(c);
    }
    message.append(" unexpected.");
    return message.toString();
  }
  public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg);
    boolean b = false;
    try {
      b = am != null && (Boolean) am.get();
    } catch (ClassCastException e) {
      b = false;
    }
    return b;
  }
  public String getString(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg);
    try {
      return am == null ? "" : (String) am.get();
    } catch (ClassCastException e) {
      return "";
    }
  }
  public int getInt(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg);
    try {
      return am == null ? 0 : (Integer) am.get();
    } catch (Exception e) {
      return 0;
    }
  }
  public boolean has(char arg) {
    return argsFound.contains(arg);
  }
  public boolean isValid() {
    return valid;
  }
  private class ArgsException extends Exception {}
  private abstract class ArgumentMarshaler {
    public abstract void set(String s) throws ArgsException;
    public abstract Object get();
  }
  private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    private boolean booleanValue = false;
    public void set(String s) {
      booleanValue = true;
    }
    public Object get() {
      return booleanValue;
    }
  }
  private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";
    public void set(String s) {
      stringValue = s;
    }
    public Object get() {
      return stringValue;
    }
  }
  private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;
    public void set(String s) throws ArgsException {
      try {
        intValue = Integer.parseInt(s);
      } catch (NumberFormatException e) {
        throw new ArgsException();
      }
    }
    public Object get() {
      return intValue;
    }
  }
}

```
</details>







함수의 형태 (외형) 을 동일하게 하면 어떤게 좋을까요? 

- factory pattern 등으로 다른형태의 함수에 대하여 동일한 동작으로 실행할 수 있음.
- (예시) 붕어빵이나, 풀빵이나, 모두 틀열기, 반죽붓기, 속에 넣기, 틀 닫기, 뒤집기, 꺼내기 로 통일할 수 있음.
- 틀의 형태(붕어냐, 국화모양이냐), 속에 뭘 넣느냐 (팥? 슈크림? 치즈? 등), 얼마나 있다가 뒤집느냐 (빨리? 느리게?) 는 종류별로 다르게 하되,
- 호출은 모두 동일하게 하여 사용 가능 (open, input, add, close, switch_side, eject)
- 변형이 필요한 부분은 interface 화 하여 override 하여 구현.
- 필요한 경우 RTTI (Run time type information) 하여 각자의 모양에 맞게 충분히 변경하여 사용 가능함.
- RTTI example : https://en.wikipedia.org/wiki/Run-time_type_information#Example_2

```c++
// factory pattern example : https://en.wikipedia.org/wiki/Factory_method_pattern#Examples

#include <iostream>
#include <memory>

enum ProductId {MINE, YOURS};

// defines the interface of objects the factory method creates.
class Product {
public:
  virtual void print() = 0;
  virtual ~Product() = default;
};

// implements the Product interface.
class ConcreteProductMINE: public Product {
public:
  void print() {
    std::cout << "this=" << this << " print MINE\n";
  }
};

// implements the Product interface.
class ConcreteProductYOURS: public Product {
public:
  void print() {
    std::cout << "this=" << this << " print YOURS\n";
  }
};

// declares the factory method, which returns an object of type Product.
class Creator {
public:
  virtual std::unique_ptr<Product> create(ProductId id) {
    if (ProductId::MINE == id) return std::make_unique<ConcreteProductMINE>();
    if (ProductId::YOURS == id) return std::make_unique<ConcreteProductYOURS>();
    // repeat for remaining products...

    return nullptr;
  }
  virtual ~Creator() = default;
};

int main() {
  // The unique_ptr prevent memory leaks.
  std::unique_ptr<Creator> creator = std::make_unique<Creator>();
  std::unique_ptr<Product> product = creator->create(ProductId::MINE);
  product->print();

  product = creator->create(ProductId::YOURS);
  product->print();
}


```

output : 
```c++
this=0x6e5e90 print MINE
this=0x6e62c0 print YOURS
```
