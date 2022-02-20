```java
public PageInfo pageQuery(CusQueryDTO cusQueryDTO) {
    PageHelper.startPage(cusQueryDTO.getPageNum(), cusQueryDTO.getPageSize());
    List<BdCrmCustomer> bdCrmCustomers = bdCrmCustomerMapper.selectAll();
    PageInfo pageInfo = new PageInfo(bdCrmCustomers);
    return pageInfo;
}
```
