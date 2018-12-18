---
layout: post
title:  "Builder Pattern"
fullview: true
comments: true
archive: true
date:   2018-12-17 20:37:00 -0500
categories: Java
tags: Builder Pattern Java
---

Java에서는 생성자를 이용하여 Object를 생성한다. 물론 이 외에도 Factory Pattern을 이용하여 객체를 생성하거나 정적 생성 함수(static factory method)를 이용하여 객체를 생성할 수도 있다. 각각 모두 장단점이 있는데 이번에는 Builder Pattern을 이용하여 객체를 생성하는 방법에 대해 정리해 보고자 한다.

객체 생성을 위해서 생성자는 필수로 필요하다. 단순히 기본 생성자 만으로 객체를 생성하는 것이라면 큰 문제가 없지만 class가 가지는 member variables이 많다면 생성자 또한 복잡해지는 경향이 있다.

```java
public class Employee {

    private String name;
    private String employeeId;
    private Date startDate;
    private Department department;
    private JobTitle jobTitle;
    private String managerId;

    public Employee(String name, String employeeId, Date startDate, Department department, JobTitle jobTitle, String managerId) {
        this.name = name;
        this.employeeId = employeeId;
        this.startDate = startDate;
        this.department = department;
        this.jobTitle = jobTitle;
        this.managerId = managerId;
    }
}

public class CreateEmployee {
    public static void main(String...args) {
        Employee john = new Employee("John", "20181217001", new Date(), Department.NOC, JobTitle.SystemEngineer, "20001220001");
        Employee james = new Employee("James", "20181217002", new Date(), Department.APIDev, JobTitle.SoftwareEngineer, "20110112015");
    }
}
```

위의 예제를 보면 john이라는 Employee객체 생성을 위해 무수히 많은 인자를 넣어 주었다. 한번에 객체를 생성할 수 있는 것은 좋지만 생성코드를 보고 어떤 인자가 무엇을 의미하는지 한눈에 알아보기가 쉽지 않다. 또한 Employee class에 member variables이 추가될 때마다 생성자의 input parameter수가 달라지거나 여러 생성자를 가져야 하는 구조이기 때문에 유지보수에도 조금 어려움이 있어 보인다.

객체 생성 시 모든 값을 한번에 전달하여 객체를 생성해야만 하는 것은 아니다. 기본 생성자를 이용하거나 필수 인자만을 가진 생성자를 이용하여 객체를 생성 한 후 나머지 인자들을 전달하는 방식으로도 객체 생성이 가능하다.

```java
@Data
public class Employee {

    private String name;
    private String employeeId;
    private Date startDate;
    private Department department;
    private JobTitle jobTitle;
    private String managerId;

    public Employee(String name, String employeeId) {
        this.name = name;
        this.employeeId = employeeId;
    }

    public Employee(String name, String employeeId, Date startDate, Department department, JobTitle jobTitle, String managerId) {
        this.name = name;
        this.employeeId = employeeId;
        this.startDate = startDate;
        this.department = department;
        this.jobTitle = jobTitle;
        this.managerId = managerId;
    }
}

public class CreateEmployee {
    public static void main(String...args) {
        Employee john = new Employee("John", "20181217001");
        john.setStartDate(new Date());
        john.setDepartment(Department.NOC);
        john.setJobTitle(JobTitle.SystemEngineer);
        john.setManagerId("20001220001");
        Employee james = new Employee("James", "20181217002");
        james.setStartDate(new Date());
        james.setDepartment(Department.APIDev);
        james.setJobTitle(JobTitle.SoftwareEngineer);
        james.setManagerId("20110112015");
    }
}
```

위의 코드를 보면 확실히 이전에 비해 한눈에 알아보기 좋은 코드로 바뀐 것 같다. 하지만 객체 생성이 한번에 이루어지지 않고 여러줄에 걸쳐 여러 setter 함수를 호출 한 후에야 객체 생성이 완료되었다.

Builder Pattern 은 이런 점을 해결해 준다. 객체 생성을 유연하게 하면서 한번의 호출로 객체를 생성할 수 있게 해준다.

```java
public interface Builder<T> {
    T build();
}

@Data
public class Employee {
    private String name;
    private String employeeId;
    private Date startDate;
    private Department department;
    private JobTitle jobTitle;
    private String managerId;

    private Employee() {}
    public EmployeeBuilder builder() {
        return new EmployeeBuilder();
    }

    public static class EmployeeBuilder implements Builder<Employee> {
        private String name;
        private String employeeId;
        private Date startDate;
        private Department department;
        private JobTitle jobTitle;
        private String managerId;

        private EmployeeBuilder() {}
        public EmployeeBuilder name(String name) {
            this.name = name;
            return this;
        }
        public EmployeeBuilder employeeId(String employeeId) {
            this.employeeId = employeeId;
            return this;
        }
        public EmployeeBuilder startFrom(Date date) {
            this.date = date;
            return this;
        }
        public EmployeeBuilder workAt(Department department) {
            this.department = department;
            return this;
        }
        public EmployeeBuilder title(JobTitle jobTitle) {
            this.jobTitle = jobTitle;
            return this;
        }
        public EmployeeBuilder manager(String managerId) {
            this.managerId = managerId;
            return this;
        }
        @Override
        public Employee build() {
            Employee employee = new Employee();
            employee.setName(this.name);
            employee.setEmployeeId(this.employeeId);
            employee.setStartDate(this.startDate);
            employee.setDepartment(this.department);
            employee.setJobTitle(this.jobTitle);
            employee.setManagerId(this.managerId);
            return employee;
        }
    }
}

public class CreateEmployee {
    public static void main(String...args) {
        Employee john = Employee.builder()
            .name("John")
            .employeeId("20181217001")
            .startFrom(new Date());
            .workAt(Department.NOC);
            .title(JobTitle.SystemEngineer);
            .managerId("20001220001")
            .build();
        Employee james = Employee.builder()
            .name("James")
            .employeeId("20181217002")
            .startFrom(new Date())
            .workAt(Department.APIDev)
            .title(JobTitle.SoftwareEngineer)
            .managerId("20110112015")
            .build();
    }
}
```

위의 코드를 보면 EmployeeBuilder class가 추가되어 좀 더 복잡해 보이지만 실제 객체를 생성하는 코드를 보면 객체를 한번에 생성하면서도 한눈에 쉽게 알아 볼 수 있도록 되어 있다. 이 패턴은 생성자의 인자 수가 많은 경우에 사용하면 객체 생성 코드가 읽기 쉽고 관리하기 쉽게 바꿀 수 있는 장점이 있다. 단점이라고 하면 위의 코드에서 보이는 것 처럼 Builder클래스가 동일한 인자를 가져야 한다는 것이 단점이라고 할 수 있겠다.