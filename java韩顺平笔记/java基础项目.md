# 零钱通项目

//主程序

```java
package com.project.pocketmoney;

import java.util.Scanner;

public class PocketMoney {
    public static void main(String[] args) {
        boolean test = true;
        income []income = new income[10000];
        int n = 0;
        double balance = 0;
        do
        {
            Scanner sc = new Scanner(System.in);
            System.out.println("=========零钱通菜单=========");
            System.out.println("请输入指令");
            int i = sc.nextInt();
            switch(i)
            {
                case 1:
                    for(int j = 1;j <= n;j++)
                    {
                        System.out.println(income[j].getSource()+"  "+income[j].getIncome()+"   " +
                                income[j].getTime() + "余额:"+balance);
                    }
                    break;
                case 2:
                case 3:
                    String scours = sc.next();
                    income[++n] = new income();
                    income[n].setSource(scours);
                    String in = sc.next();
                    income[n].setIncome(in);
                    String time = sc.next();
                    income[n].setTime(time);
                    int t = Integer.parseInt(in);
                    balance += t;
                    //消费
                    break;
                case 4:
                    //退出
                    test = false;
                    break;
            }
        }while(test);
    }
}

```

//    类实现

```java
package com.project.pocketmoney;

public class income {
    String incomes;
    private String time;

    public String getSource() {
        return source;
    }

    public income() {
    }

    public void setSource(String source) {
        this.source = source;
    }

    public String getTime() {
        return time;
    }

    public void setTime(String time) {
        this.time = time;
    }

    public String getIncome() {
        return incomes;
    }

    public void setIncome(String income) {
        this.incomes = income;
    }

    private String source;

    public income(String income, String time, String source) {
        this.incomes = income;
        this.time = time;
        this.source = source;
    }
}

```

四种访问修饰符和各自的访问权限

1. public 公共  子类,本类,同一个包下的类,不同包下的类都可以使用
2. protected   受保护的    子类,本类,同一个包下的类可以使用,不同包下的类不能使用
3. 默认   子类本类

# 房屋出租

1. 添加房屋信息
2. 查看房屋信息
3. 删除房屋信息
4. 修改房屋信息
5. 
