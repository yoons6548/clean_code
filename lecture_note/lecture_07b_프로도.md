## 호출자를 고려해 예외 클래스를 정의하라


```java
// try - catch 로 계속 감싸는 것은 코드가 지저분해짐. (하지만 우리가 흔히 쓰는 방법)


ACMEPort port = new ACMEPort(12);
try {
    port.open();
} catch (DeviceResponseException e) {                  // 처리가
    reportPortError(e);
    logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {                 // 정말
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) {                                 // 많아요
    reportPortError(e);
    logger.log("Device response exception");
} finally {…}
```

```java
// 호출부에는 최소한의 관련 error 만 처리하여 깨끗이 처리하기. 
LocalPort port = new LocalPort(12);
try {
    port.open();
} catch (PortDeviceFailure e) { // 정말 필요한것만 호출시에 쓰고 
    reportError(e);
    logger.log(e.getMessage(), e);
} finally {…}


// 호출시에 종속성을 줄일 수 있는 방법
public class LocalPort {
    private ACMEPort innerPort;
    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }
    public void open() {
        try {
            innerPort.open();
        } catch (DeviceResponseException e) {  // 미리 
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) { // 정의할 수 있는건 
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {                 // 선언에 미리 쓰기
            throw new PortDeviceFailure(e);
        }
    }…
}
```

## 정상 흐름을 정의하라

```java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch (MealExpensesNotFound e) { // expenseReportDAO.getMeals 가 항상 return 보장하면 필요 없음
    m_total += getMealPerDiem();
}
```

```java
public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        // return the per diem default
    }
}

// getMeals 가 return 을 보장할 경우 예외처리는 없앨 수도 있음. 
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```
- 입출력을 고려 안하고 작성하는 경우가 많은데, 함수의 명세 (I/O)를 잘 고려하여 작성하면 오류도 줄이고, 깔끔한 코드를 작성할 수가 있음.  

## null을 반환하지 마라
```java
// 1. 빠져버린 null check
public void registerItem(Item item) {
    if (item != null) {
        ItemRegistry registry = peristentStore.getItemRegistry();
        if (registry != null) {
            Item existing = registry.getItem(item.getID());
            if (existing.getBillingPeriod().hasRetailOwner()) { // null check 빠짐
                existing.register(item);
            }
        }
    }
}
```
- 너무 많은 null check 가 있으니 빠질 수 있음. 
1. null check 보다 list 를 이용하면 null 일 경우 처리 않고 지나갈 수 있음. 
2. null 대신 emptyList() 를 반환하면 null 로 인한 오류를 사전에 방지할 수 있다. 

```java
List < Employee > employees = getEmployees();
if (..there are no employees..)
    return Collections.emptyList();
for (Employee e: employees) {
    totalPay += e.getPay();
}
```

## null을 전달하지 마라
```java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        return (p2.x– p1.x) * 1.5;
    }…
}

calculator.xProjection(null, new Point(12, 13)); // NPE!!
```

```java
// 1. null check 를 해서 throw 하는 방법
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        if (p1 == null || p2 == null) { // 방법 1
            throw InvalidArgumentException(
                "Invalid argument for MetricsCalculator.xProjection");
        }
        return (p2.x– p1.x) * 1.5;
    }
}
// 2. assert 로 check 하는 방법
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        assert p1 != null: "p1 should not be null"; // 방법 2
        assert p2 != null: "p2 should not be null";
        return (p2.x– p1.x) * 1.5;
    }
}
```

* 결론

- 제발 함수 명세 잘 보고, 
- 미리 정의할 수 있는 에러는 정의 잘 하고, 
- 효율적으로 작성하면 좋겠습니다. :) 
