> ### Single Responsibility Principle

SOLID 원칙 중 의미 전달이 되지 못한 원칙은 바로 단일 책임 원칙이다. <br>
-> 부적절한 이름 때문이기도 하다.

프로그래머가 이름만 들었을 때, 모든 모듈이 단 하나의 일만 해야 한다는 의미로 받아드려지기 쉽다.

프로그래머는 단 하나의 일만 해야 한다는 것에 몰두되어 혼동하기 쉽다.

사실 단 하나의 일만 해야 한다는 원칙은 함수에서 사용되는데, 커다란 함수들을
작은 함수들로 리팩토링하는 더 저수준에서 사용된다.

위 원칙은 SOLID도 아니며, SRP도 아니다!

저자가 첫번째로 설명하는 SRP란 '하나의 모듈은 하나의, 오직 하나의 사용자 또는 이해관계자에 대해서만 책임져야 한다.' 이다. <br>
하지만 여기서 문제점이 있는데, 시스템이 동일한 방식으로 변경되기를 원하는 사용자와 이해관계자는 두 명 이상일 수도 있기 때문에,
해당 변경을 요청하는 한 명 이상의 사람 즉 집단을 뜻한다.

최종적으로 설명하는 SRP란 다음과 같다. **'하나의 모듈은 하나의, 오직 하나의 액터에 대해서만 책임져야 한다.'** <br>
-> 모듈이란 '소스 파일/단순히 함수와 데이터 구조로 구성된 응집된 결합' 이다.

사실 책에서 저자의 말만 읽고, 이해가 되지 않아 조금 더 이해해보고자 사진에 대해 코드로 작성해보았다.

### 잘못된 사레, 1. 우발적 중복
![Employee Class.png](asset%2FEmployee%20Class.png)
위 그림의 Employee 클래스는 SRP를 위반하는데, 세가지 메소드가 서로 매우 다른 액터를 책임지기 때문이다.
<pre>
<code>
class Employee {
    private String name;
    private int id;

    public double calculatePay(int hours) {
        // 회계팀이 CFO에게 보고
        return 1000.0 * hours;
    }
    public String reportHours(int totalHours) {
        // 인사팀이 COO에게 보고
        return totalHours + "시간 근무했습니다.";
    }
    public void save() {
        // DBA가 DB에 저장하고, CTO에게 보고
    }
</code>
</pre>

이러한 세가지 액터가 결합되면서, CFO팀에서 결정한 조치가 COO팀에 영향을 줄 수 있다는 것이다. <br>
예를 들어, calculatePay() 메소드와 reportHours() 메소드가 초과 근무를 제외한 업무 시간을 계산하는 알고리즘을 공유하고, <br>
개발자는 코드 중복을 피하기 위해 regularHours()라는 메소드에 넣었다고 가정한다.

<pre>
<code>
class Employee {
    private String name;
    private int id;

    // 정규 근무 시간을 계산하는 메서드 (중복을 피하기 위해 공통 로직으로 분리됨)
    private int regularHours(int totalHours) {
        // 초과 근무를 제외한 정규 근무 시간 계산
        return totalHours > 40 ? 40 : totalHours;
    }

    // 급여 계산 메서드 (회계팀, CFO 보고)
    public double calculatePay(int totalHours) {
        int regularWorkHours = regularHours(totalHours);
        return 1000.0 * regularWorkHours;
    }

    // 근무 시간 보고 메서드 (인사팀, COO 보고)
    public String reportHours(int totalHours) {
        int regularWorkHours = regularHours(totalHours);
        return regularWorkHours + " 시간 근무했습니다.";
    }

    // 직원 정보 저장 (DBA, CTO 보고)
    public void save() {
        // 직원 정보를 DB에 저장하는 로직
        System.out.println("직원 정보를 저장했습니다.");
    }
}
</code>
</pre>

CFO 팀은 새로운 메소드가 원하는 방식으로 동작하는 검증하고, 시스템에 배포를 하고, 
COO 직원은 reportHours() 메소드가 생성한 보고서를 여전히 이용한다면 잘못된 데이터로 인해 수백만 달러의 예산이 지출되는 문제가 발생할 수 있다.

이와 같은 문제 때무네 SRP는 서로 다른 애거가 의존하는 코드를 서로 분리하라고 말한다!

### 잘못된 사례, 2. 병합
병합의 사례는 일반 프로젝트를 진행하면서도, 자주 발생하는 문제이다. 예를 들어, DBA가 속한 CTO 팀에서 데이터베이스의 Employee 테이블 스키마를
수정하기로 결정했고, COO 팀에서는 reportHours() 메소드의 포맷을 변경하기로 했다.

두 명의 서로 다른 개발자가, 서로 다른 팀에 속했을 때 Employee 클래스를 체크아웃 받은 후 변경사항을 적용하기로 했다. 안타깝게도 이들의 변경사항은
서로 충돌하게 된다. 최근 도구들이 굉장히 빠른 속도로 발전하지만, 어떤 도구도 병합이 발생하는 모든 경우를 해결해주진 않는다.

이러한 문제를 벗어나는 방법은 서로 다른 액터를 뒷받침하는 코드를 서로 분리하는 것이다!


### 해결책
우발적 중복과 병합을 해결하는 문제는, 메소드를 각기 다른 클래스로 이동하면 해결된다.
![세 클래스는 서로의 존재를 알지 못한다..png](asset%2F%EC%84%B8%20%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94%20%EC%84%9C%EB%A1%9C%EC%9D%98%20%EC%A1%B4%EC%9E%AC%EB%A5%BC%20%EC%95%8C%EC%A7%80%20%EB%AA%BB%ED%95%9C%EB%8B%A4..png)

<pre>
<code>
class EmployeeData {
    private String name;
    private int hoursWorked;
    private double salary;

    public EmployeeData(String name, int hoursWorked, double salary) {
        this.name = name;
        this.hoursWorked = hoursWorked;
        this.salary = salary;
    }

    // Getters and setters
    public String getName() {
        return name;
    }

    public int getHoursWorked() {
        return hoursWorked;
    }

    public double getSalary() {
        return salary;
    }
}

class PayCalculator {
    public double calculatePay(EmployeeData employeeData) {
        return employeeData.getSalary() * employeeData.getHoursWorked();
    }
}

class HourReporter {
    public void reportHours(EmployeeData employeeData) {
        System.out.println(employeeData.getName() + "님의 근무 시간은 " + employeeData.getHoursWorked() + "입니다.");
    }
}

class EmployeeSaver {
    public void saveEmployee(EmployeeData employeeData) {
        System.out.println(employeeData.getName() + "님의 정보를 저장했습니다.");
    }
}
</code>
</pre>

![퍼사드 패턴.png](asset%2F%ED%8D%BC%EC%82%AC%EB%93%9C%20%ED%8C%A8%ED%84%B4.png)

<pre>
<code>
class EmployeeFacade {
    private PayCalculator payCalculator;
    private HourReporter hourReporter;
    private EmployeeSaver employeeSaver;

    public EmployeeFacade() {
        this.payCalculator = new PayCalculator();
        this.hourReporter = new HourReporter();
        this.employeeSaver = new EmployeeSaver();
    }

    public void calculatePay(EmployeeData employeeData) {
        double pay = payCalculator.calculatePay(employeeData);
    }

    public void reportHours(EmployeeData employeeData) {
        hourReporter.reportHours(employeeData);
    }

    public void saveEmployee(EmployeeData employeeData) {
        employeeSaver.saveEmployee(employeeData);
    }
}
</code>
</pre>

![가장 중요한 메소드를 빼고, 퍼사드 패턴.png](asset%2F%EA%B0%80%EC%9E%A5%20%EC%A4%91%EC%9A%94%ED%95%9C%20%EB%A9%94%EC%86%8C%EB%93%9C%EB%A5%BC%20%EB%B9%BC%EA%B3%A0%2C%20%ED%8D%BC%EC%82%AC%EB%93%9C%20%ED%8C%A8%ED%84%B4.png)

<pre>
<code>
class Employee {
    private EmployeeData employeeData;
    private HourReporter hourReporter;
    private EmployeeSaver employeeSaver;

    public Employee(EmployeeData employeeData) {
        this.employeeData = employeeData;
        this.hourReporter = new HourReporter();
        this.employeeSaver = new EmployeeSaver();
    }

    public void calculatePay() {
        PayCalculator payCalculator = new PayCalculator();
        double pay = payCalculator.calculatePay(employeeData);
    }

    public void reportHours() {
        hourReporter.reportHours(employeeData);
    }

    public void save() {
        employeeSaver.saveEmployee(employeeData);
    }
}
</code>
</pre>
