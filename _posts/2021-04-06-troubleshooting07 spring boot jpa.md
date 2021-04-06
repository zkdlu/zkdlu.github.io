---
layout: post
title: "[Trouble shooting] 6.Spring data jpa"
description: "JPA를 사용하면서 발생한 예외"
date: 2021-04-06 00:00:00
tags: [trouble shooting, spring boot, jpa]
comments: true
share: true
---

### 환경

- OpenJDK 11

- Spring Boot 2.3.8
- org.springframework.boot:spring-boot-starter-data-jpa
- com.h2database:

![jpa](https://zkdlu.github.io/images/troubleshooting/jpa.png)



초기화 sql

```sql
insert into shops (shop_id, name, min_price, open) values ('shop1', '건s shop', 10000, true);
insert into menus (menu_id, name, price, shop_id) values ('menu1', '만두', 3000, 'shop1');
insert into menus (menu_id, name, price, shop_id) values ('menu2', '라면', 4000, 'shop1');
insert into menus (menu_id, name, price, shop_id) values ('menu3', '고기', 3000, 'shop1');
insert into menus (menu_id, name, price, shop_id) values ('menu4', '볶음밥', 2000, 'shop1');
insert into menus (menu_id, name, price, shop_id) values ('menu5', '곱창', 5000, 'shop1');
```



### 무엇을 했는가?

OrderService에서 OrderDTO 객체가 넘어오면 OrderDTO에 있는 데이터로 List<OrderItem>을 만들고 Order 객체를 생성한다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
@Table(name = "ORDERS")
public class Order {
    public enum OrderState { 결제대기, 수락대기, 조리중, 배송중, 완료, 취소 }

    @Id
    @Column(name = "ORDER_ID")
    private String id;

    @ManyToOne
    @JoinColumn(name = "SHOP_ID")
    private Shop shop;

    @OneToMany
    @JoinColumn(name = "ORDER_ID")
    private List<OrderItem> orderItems = new ArrayList<>();

    @Enumerated(EnumType.STRING)
    @Column(name = "ORDER_STATE")
    private OrderState state;

    @Builder
    public Order(String id, Shop shop, List<OrderItem> orderItems) {
        this.id = id;
        this.shop = shop;
        this.orderItems = orderItems;
    }
    
    .. 메서드 ..
}

@Getter
@NoArgsConstructor
@Entity
@Table(name = "ORDERITEMS")
public class OrderItem {
    @Id
    @Column(name = "ORDER_ITEM_ID")
    private String id;
    @Column(name = "ORDER_ITEM_NAME")
    private String name;
    @ManyToOne
    @JoinColumn(name = "MENU_ID")
    private Menu menu;
    @Column(name = "COUNT")
    private int count;

    public OrderItem(Menu menu) {
        this.id = menu.getId();
        this.name = menu.getName();
        this.menu = menu;
        this.count = 1;
    }
    
    ... 메서드 ...
}
```



### 어떤일이 있었는가?

주문 객체를 생성까진 잘 되었는데 영속화를 시키는 과정에서 예외가 발생하였다.

```java
@Transactional
public String placeOrder(OrderDto orderDto) {
    Shop shop = shopRepository.findById(orderDto.getShopId())
        .orElseThrow(IllegalStateException::new);

    var orderItems = shop.getMenus().stream()
        .map(OrderItem::new)
        .collect(Collectors.toList());

    Order order = Order.builder()
        .id(orderDto.getId())
        .shop(shop)
        .orderItems(orderItems)
        .build();

    order.place();
    orderRepository.save(order);  // 예외가 발생한 부분

    return order.getId();
}
```

> 토큰을 생성하는 코드



```bash
javax.persistence.EntityNotFoundException: Unable to find com.zkdlu.tdd.domain.order.OrderItem with id menu1
	at org.hibernate.jpa.boot.internal.EntityManagerFactoryBuilderImpl$JpaEntityNotFoundDelegate.handleEntityNotFound(EntityManagerFactoryBuilderImpl.java:163) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultLoadEventListener.load(DefaultLoadEventListener.java:216) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultLoadEventListener.proxyOrLoad(DefaultLoadEventListener.java:332) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultLoadEventListener.doOnLoad(DefaultLoadEventListener.java:108) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultLoadEventListener.onLoad(DefaultLoadEventListener.java:74) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	....
```

> 발생한 예외 메시지

"menu1"이라는 OrderItem 엔티티가 없다면서 예외가 발생하였다.



### 어떻게 하였는가?

Order에서 OrderItem를 참조할 수 있는 1:N 관계지만 잘 생각해보면 1:N 관계에서 OrderItem 테이블의 N개의 컬럼이 Order 테이블 1개의 컬럼에 매칭 되려면 별도의 테이블을 두지 않는다면 결국 참조키는 OrderItem에 있어야만 한다. 

그렇다면 참조키를 가진 엔티티가 주인이 되고 그 반대는 데이터 조회만 가능한데, 현재 OrderItem은 아직 영속 상태가 아니기때문에 JPA에서는 "menu1"이라는 엔티티가 없다고 하는 것이다.

영속 상태로 만드는 가장 쉬운 방법은 Repository를 만들면 되지만,  나는 Order가 Root Aggregate이기 때문에 이는 좋지 않은 방법인것 같다.



다행스럽게도 JPA에서는 손 쉽게 영속성을 전이 시킬 수 있는 속성을 제공한다!

```java
 public class Order {
    ....생략
 
    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "ORDER_ID")
    private List<OrderItem> orderItems = new ArrayList<>();
    
    
    ....생략
}
```
