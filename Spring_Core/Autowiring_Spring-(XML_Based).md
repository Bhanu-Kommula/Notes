So, till now we were injecting the dependencies using **constructor** or **setter injections**,  
but we can also directly ask the Spring container to automatically inject the dependencies using the **`autowire`** attribute in the `<bean>` tag.

So instead of manually using `<constructor-arg>` or `<property>`, we can tell Spring to do it for us based on matching **name**, **type**, or **constructor**.

There are mainly **3 common autowiring modes**:


`autowire="byName"`

- Spring checks the bean's property name.
- If there’s another bean with the **same ID as the property name**, Spring will inject that bean automatically.

 *So here, the name of the property and the bean ID in the XML should be the same.*

       
      <bean id="employee" class="com.myproject.Employee" autowire="byName"/>
      <bean id="address" class="com.myproject.Address"/>

If Employee class has a property named address, and there is a bean with id="address", Spring will inject it automatically.


`autowire="byType"`

Spring checks the type of the property.

If there’s exactly one bean of that type, Spring will inject it.

This will throw an error if multiple beans of the same type exist — Spring won’t know which one to inject.


     <bean id="employee" class="com.myproject.Employee" autowire="byType"/>
     <bean id="addressBean" class="com.myproject.Address"/>


Here, even though the id is different, Spring will inject it because the type matches Address.


`autowire="constructor"`

This works like byType, but applies to constructor arguments instead of properties.

Spring looks at the constructor parameters and tries to match them by type from the available beans.

     <bean id="employee" class="com.myproject.Employee" autowire="constructor"/>
     <bean id="address" class="com.myproject.Address"/>


If the Employee constructor takes an Address object, Spring will automatically inject the matching bean.


Next Concept - https://github.com/Bhanu-Kommula/Notes/blob/2433fabe5a3cf289f479c57533aeeeb2509892d6/Spring_Core/Examples%20-%20Autowiring_Spring-(XML_Based).md
Previous Concept - https://github.com/Bhanu-Kommula/Notes/blob/15b269c6175c5541f606d7c1167748fa64e33099/Spring_Core/XML-Based-Config.md
