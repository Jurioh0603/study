## 스프링 로직
Controller -> interfaceService -> interfaceDAO -> DB -> Controller
구현 :        ServiceImpl         DAOImpl

## mybatis 로직
Controller -> interfaceService -> Mapper -> DB -> Controller
구현 :        ServiceImpl         .xml
