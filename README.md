Springboot + Vue Front-Back Separation - Warehouse Management System
Building a Front-Back Separation Project from Scratch


Project Overview
Backend: Springboot, MyBatis-Plus, Java
Frontend: Node.js, Vue CLI, Element-UI
Database: MySQL

I. Creating the Backend Project
Create a folder.
Open the folder in IntelliJ IDEA.
Create a new module (Springboot).
Create a test class and run tests.
II. Adding MyBatis-Plus Support
Add dependency code.
Create a database instance, create a user table, and insert default data.

xml
Copy
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.1</version>
</dependency>
sql
Copy
create table user
(
    id int auto_increment comment 'Primary Key'
    primary key,
    no varchar(20) null comment 'Account',
    name varchar(100) not null comment 'Name',
    password varchar(20) not null comment 'Password',
    age int null,
    sex int null comment 'Gender',
    phone varchar(20) null comment 'Phone',
    role_id int null comment 'Role 0 Super Admin, 1 Admin, 2 Regular Account',
    is_valid varchar(4) default 'Y' null comment 'Whether valid, Y for valid, others for invalid'
)
charset = utf8;
Configuration in the yml file (port and data source configuration).
Write test code.
Create entity class:
java
Copy
@Data
public class User {
    private int id;
    private String no;
    private String name;
    private String password;
    private int sex;
    private int roleId;
    private String phone;
    private String isValid;
}
Create mapper interface:
java
Copy
@Mapper
public interface UserMapper extends BaseMapper<User> {
    public List<User> selectAll();
}
Create service interface and implementation class:
java
Copy
public interface UserService extends IService<User> {
    public List<User> selectAll();
}

@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    @Resource
    private UserMapper userMapper;

    @Override
    public List<User> selectAll() {
        return userMapper.selectAll();
    }
}
Create mapper XML file:
xml
Copy
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.wms.mapper.UserMapper">
    <select id="selectAll" resultType="com.wms.entity.User">
        select * from user
    </select>
</mapper>
Create controller:
java
Copy
@RestController
public class HelloController {
    @Autowired
    private UserService userService;

    @GetMapping
    public List<User> hello() {
        return userService.selectAll();
    }
}
III. Using the Code Generator to Generate Code
Add dependencies:
xml
Copy
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.30</version>
</dependency>
<dependency>
    <groupId>com.spring4all</groupId>
    <artifactId>spring-boot-starter-swagger</artifactId>
    <version>1.5.1.RELEASE</version>
</dependency>
Get the generator code:
java
Copy
package com.wms.common;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class CodeGenerator {
    /**
     * <p>
     * Read console input
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("Please input " + tip + ":");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("Please input the correct " + tip + "!");
    }

    /**
     * Operation steps:
     * 1. Modify the data source including address and password information, corresponding code marked as: I, same below
     * 2. Module configuration, can modify package name
     * 3. Modify template (this step can be ignored)
     * @param args
     */
    public static void main(String[] args) {
        // Code generator
        AutoGenerator mpg = new AutoGenerator();
        // Global configuration
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir") + "/wms";
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("wms");
        gc.setOpen(false);
        gc.setSwagger2(true); // Entity attribute Swagger2 annotation
        gc.setBaseResultMap(true); // XML ResultMap
        gc.setBaseColumnList(true); // XML columList
        // Remove the first letter of the service interface, such as DO for User, it is called UserService
        gc.setServiceName("%sService");
        mpg.setGlobalConfig(gc);
        // Data source configuration
        DataSourceConfig dsc = new DataSourceConfig();
        // I. Modify the data source
        dsc.setUrl("jdbc:mysql://localhost:3306/wms01?useUnicode=true&characterEncoding=UTF8&useSSL=false");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("root");
        mpg.setDataSource(dsc);
        // Package configuration
        PackageConfig pc = new PackageConfig();
        // pc.setModuleName(scanner("Module Name"));
        // II. Module configuration
        pc.setParent("com.wms")
            .setEntity("entity")
            .setMapper("mapper")
            .setService("service")
            .setServiceImpl("service.impl")
            .setController("controller");
        mpg.setPackageInfo(pc);
        // Custom configuration
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };
        // If the template engine is freemarker
        String templatePath = "templates/mapper.xml.ftl";
        // If the template engine is velocity
        // String templatePath = "/templates/mapper.xml.vm";
        // Custom output configuration
        List<FileOutConfig> focList = new ArrayList<>();
        // Custom configuration will be output first
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // Custom output file name, if you set the prefix and suffix of the entity, note that the name of the xml will also change accordingly!!
                return projectPath + "/src/main/resources/mapper/" + pc.getModuleName() + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        /*
        cfg.setFileCreate(new IFileCreate() {
            @Override
            public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
                // Determine whether the custom folder needs to be created
                checkDir("Call the default method to create the directory, custom directory use");
                if (fileType == FileType.MAPPER) {
                    // Already generated mapper file, judge whether it exists, do not regenerate if you do not want to
                    return !new File(filePath).exists();
                }
                // Allow template file generation
                return true;
            }
        });
        */
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);
        // Template configuration
        TemplateConfig templateConfig = new TemplateConfig();
        // Configure custom output template
        // Specify the custom template path, do not bring .ftl/.vm, the template engine will automatically recognize
        // III. Modify the template
        /*templateConfig.setEntity("templates/entity2.java");
        templateConfig.setService("templates/service2.java");
        templateConfig.setController("templates/controller2.java");
        templateConfig.setMapper("templates/mapper2.java");
        templateConfig.setServiceImpl("templates/serviceimpl2.java");*/
        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);
        // Strategy configuration
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        // strategy.setSuperEntityClass("Your own parent entity, no need to set if none!");
        //strategy.setSuperEntityClass("BaseEntity");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // Parent class
        //strategy.setSuperControllerClass("BaseController");
        // strategy.setSuperControllerClass("Your own parent controller, no need to set if none!");
        // Public parent class fields
        // strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("Table name, multiple English commas separated").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        //strategy.setTablePrefix(pc.getModuleName() + "_");
        // Ignore table prefix tb_, for example tb_user, directly map to user object
        // IV. Note whether to remove table prefix
        //strategy.setTablePrefix("tb_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
}
Generate code and test.
IV. Implementing CRUD Operations
V. Handling Pagination
Input parameter encapsulation:
Add pagination interceptor:
Write pagination mapper method:
Include IPage as a parameter:
Custom SQL using Wrapper:
Refer to the official documentation.
VI. Data Encapsulation for Frontend
Encapsulate data to provide a unified format for the frontend to handle:
JSON
Copy
{
    "Code": 200, // 400
    "Msg": "Success, Failure",
    "Total": 10,
    "Data": []
}
VII. Creating the Frontend Project
Need to install Node.js.
VIII. Importing and Running the Vue Project in IDEA
IX. Importing Element-UI
Supplement: Vue CLI (Note version conflicts) npm install -g @vue/cli
Installation command:
bash
Copy
npm i element-ui -S
Global import in main.js:
JavaScript
Copy
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
Vue.use(ElementUI);
Test if the import is successful.
X. Building the Page Layout
Container layout container.
XI. Splitting the Page Layout
Benefits of decomposition.
XII. Writing the Header Page
Dropdown
Menu collapse icon
Welcome text
Remove background, add bottom border
XIII. Writing the Menu Navigation Page
First-level menu.
XIV. Collapsing the Menu Navigation Page
Collapse logic:
Header click icon -- submit --> Parent component -- change --> Aside child component (collapse).
XV. Installing Axios and Handling CORS
Install Axios:
bash
Copy
npm install axios --save
Global import in main.js:
JavaScript
Copy
import axios from 'axios';
Vue.prototype.$axios = axios;
CORS handling:
JavaScript
Copy
this.$axios.get('http://localhost:8090/list').then(res => {
    console.log(res);
});
this.$axios.post('http://localhost:8091/user/listP', {}).then(res => {
    console.log(res);
});
XVI. List Display
List data
Convert columns using tag
Set table header style using header-cell-style
Add borders
Buttons (Edit, Delete)
Backend return result encapsulation (Result)
XVII. Pagination Handling
Add pagination code to the page
Modify query method and parameters
Handle pagination logic (Note one issue)
XVIII. Query Handling
Query layout (including query and reset buttons)
Input box
Dropdown
Enter event (query) @keyup.enter.native
Reset handling
XIX. Adding New Entries
Add button
Pop-up window
Write the form
Submit data (Prompt information, list refresh)
Data validation
Account uniqueness validation
Form reset
XX. Editing Entries
Pass data to the form
Submit data to the backend
Form reset
XXI. Deleting Entries
Get data (id)
Confirm deletion
Submit to the backend
XXII. Login
Login page
Backend query code
Login page route
Install routing plugin (npm i vue-router@3.5.4)
Create routing file
Register in main.js
Home page route
XXIII. Logout
Display name
Logout event
Logout redirect, clear related data
Confirm logout
XXIV. Personal Center on the Home Page
Write the page
Route jump
Route error resolution
XXV. Menu Jump
Add router to the menu, highlight
Configure submenus
Simulate dynamic menu
XXVI. Dynamic Routing
Design menu table and data
Generate corresponding backend code for menu
Return data
Vuex state management
Install (npm i vuex@3.0.0)
Write store
Register in main.js
Generate menu data
Generate routing data
Get routing list
Assemble routes
JavaScript
Copy
const VueRouterPush = VueRouter.prototype.push;
VueRouter.prototype.push = function push(to) {
    return VueRouterPush.call(this, to).catch(err => err);
};
import vue from 'vue';
import Vuex from 'vuex';
vue.use(Vuex);
import store from "./store";
Routing list router.options.routes
Merge routes
Error handling
XXVII. Admin Management, User Management
XXVIII. Warehouse Management
Table design
Generate backend code based on the table
Write backend CRUD code
Test query code with Postman
Write related frontend code
XXIX. Item Type Management
Table design
Generate backend code based on the table
JavaScript
Copy
routes.forEach((routeItem) => {
    if (routeItem.path == '/Index') {
        menuList.forEach(menu => {
            let newRoute = {
                path: '/' + menu.menuclick,
                name: menu.menuname,
                meta: {
                    title: menu.menuname
                },
                component: () => import('../components/' + menu.menucomponent)
            };
            routeItem.children.push(newRoute);
        });
    }
});
router.addRoutes(routes);
export function resetRouter() {
    router.matcher = new VueRouter({
        mode: 'history',
        routes: []
    }).matcher;
}
Write backend CRUD code
Test query code with Postman
Write related frontend code
XXX. Item Management
Table design
Generate backend code based on the table
Write backend CRUD code
Test query code with Postman
Write related frontend code
Display lists of warehouses and categories
Add warehouse and category conditions to query criteria
Implement dropdowns for warehouses and categories in the form
XXXI. Record Management
Table design
Generate backend code based on the table
Write backend query code
Write related frontend code
Optimization
Display item name, warehouse, and category name in the list
Query by item name, warehouse, and category
XXXII. Inbound and Outbound Operations
Form writing
Inbound operation (Record, update item quantity, auto-fill time)
JavaScript
Copy
count: [
    { required: true, message: 'Please enter the quantity', trigger: 'blur' },
    { pattern: /^([1-9][0-9]*){1,4}$/, message: 'Quantity must be a positive integer', trigger: "blur" },
    { validator: checkCount, trigger: 'blur' }
],
let checkCount = (rule, value, callback) => {
    if (value > 9999) {
        callback(new Error('Quantity input is too large'));
    } else {
        callback();
    }
};
User selection
XXXIII. Optimization
Inbound and outbound permission control
Record query permission control
