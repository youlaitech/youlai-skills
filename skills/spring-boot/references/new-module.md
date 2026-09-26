# 新增业务资源或模块

本文件提供 youlai-boot 的目标模板。先检查同一业务域现有代码和数据库约定；不要为了套模板创建无意义的层。

以下以 system 域新增“通知公告”资源为例。

## 1. 确定契约

先明确：

- 创建、更新字段是否相同；

- 哪些字段由服务器生成，哪些允许客户端修改；

- 列表是否分页、允许哪些排序字段；

- 是否需要联表投影、数据权限、逻辑删除或多表事务；

- 创建成功是否需要返回 ID；

- 需要哪些权限、审计和幂等保护。

## 2. 目录

```text
system/
├── controller/NoticeController.java
├── converter/NoticeConverter.java
├── mapper/NoticeMapper.java
├── model/
│   ├── entity/Notice.java
│   ├── form/NoticeCreateForm.java
│   ├── form/NoticeUpdateForm.java
│   ├── query/NoticePageQuery.java
│   └── vo/NoticePageVO.java
└── service/
    ├── NoticeService.java
    └── impl/NoticeServiceImpl.java
```

如果资源属于新的独立业务域，再创建顶级业务包；不要把普通资源一律称为“模块”。

模块自带配置时：属性类放 `property/` 子包（业务模块根是集合根），模块私有 Bean 的装配类放 `config/`，仅装配跨域 Bean 才进全局 `config/`。`model/` 根只保留分类子包（dto/entity/form/query/vo），跨层载体进 `dto/` 并按包内 `XxxDTO` 后缀命名，不在根上落散文件。

## 3. Entity

```java
@Data
@EqualsAndHashCode(callSuper = true)
@TableName("sys_notice")
public class Notice extends BaseEntity {

    private String title;

    private String content;

    private Integer type;

    private Integer status;

    private Integer isDeleted;
}
```

- Java 类型使用领域名，表前缀由 `@TableName` 表达。

- ID 和逻辑删除策略以仓库的 MyBatis-Plus 全局配置为准。

- Entity 不添加仅供前端展示的联表字段，不从 Controller 返回。

## 4. Form

创建和更新约束不同时拆分：

```java
@Data
@Schema(description = "通知公告创建参数")
public class NoticeCreateForm {

    @NotBlank(message = "通知标题不能为空")
    @Size(max = 50, message = "通知标题长度不能超过50个字符")
    private String title;

    @NotBlank(message = "通知内容不能为空")
    private String content;

    @NotNull(message = "通知类型不能为空")
    private Integer type;
}

@Data
@Schema(description = "通知公告更新参数")
public class NoticeUpdateForm {

    @NotBlank(message = "通知标题不能为空")
    @Size(max = 50, message = "通知标题长度不能超过50个字符")
    private String title;

    @NotBlank(message = "通知内容不能为空")
    private String content;

    @NotNull(message = "通知类型不能为空")
    private Integer type;
}
```

Form 不携带 ID；更新目标由 Path Variable 唯一确定。如果创建和更新字段完全一致，可使用一个 `NoticeForm`，但需在命名或注释中说明其用途。

## 5. Query 与 VO

```java
@Data
@EqualsAndHashCode(callSuper = true)
@Schema(description = "通知公告分页查询")
public class NoticePageQuery extends BaseQuery {

    private String keywords;

    private Integer type;
}

@Data
@Schema(description = "通知公告分页项")
public class NoticePageVO {

    private Long id;

    private String title;

    private Integer type;

    private Integer status;

    private LocalDateTime createTime;
}
```

- 分页模型以 `PageQuery` 结尾。

- 时间区间使用 `createdFrom`、`createdTo` 等类型明确的字段。

- 只给客户端可以控制的查询条件建字段；数据权限上下文由安全层提供。

## 6. Mapper 与 Converter

单表 CRUD 使用 Entity + MapStruct：

```java
@Mapper
public interface NoticeMapper extends BaseMapper<Notice> {
}

@Mapper(componentModel = "spring")
public interface NoticeConverter {

    Notice toEntity(NoticeCreateForm form);

    @Mapping(target = "id", ignore = true)
    void updateEntity(NoticeUpdateForm form, @MappingTarget Notice entity);

    NoticePageVO toPageVO(Notice entity);
}
```

复杂联表分页可以让 Mapper 直接返回 `IPage<NoticePageVO>`，无需先构造完整 Entity；Mapper 不接收 Form。

## 7. Service

业务接口不要继承 `IService<Notice>`：

```java
public interface NoticeService {

    IPage<NoticePageVO> getNoticePage(NoticePageQuery query);

    Long createNotice(NoticeCreateForm form);

    void updateNotice(Long noticeId, NoticeUpdateForm form);

    void deleteNotices(List<Long> noticeIds);
}
```

```java
@Service
@RequiredArgsConstructor
public class NoticeServiceImpl implements NoticeService {

    private final NoticeMapper noticeMapper;
    private final NoticeConverter noticeConverter;

    @Override
    public Long createNotice(NoticeCreateForm form) {
        Notice entity = noticeConverter.toEntity(form);
        noticeMapper.insert(entity);
        return entity.getId();
    }

    @Override
    public void updateNotice(Long noticeId, NoticeUpdateForm form) {
        Notice entity = requireNotice(noticeId);
        noticeConverter.updateEntity(form, entity);
        noticeMapper.updateById(entity);
    }
}
```

- Service 暴露用例，不暴露任意 `getOne/list/saveOrUpdate`。

- 单条 SQL 不需要为了模板机械加事务；多表一致写入时再定义事务边界。

- 不返回 `Result`；分页契约在同一模块保持一致。

## 8. Controller

```java
@Tag(name = "08.通知公告")
@Validated
@RestController
@RequestMapping("/api/v1/notices")
@RequiredArgsConstructor
public class NoticeController {

    private final NoticeService noticeService;

    @Operation(summary = "通知分页列表")
    @GetMapping
    @PreAuthorize("@ss.hasPerm('sys:notice:list')")
    public PageResult<NoticePageVO> getNoticePage(@Valid NoticePageQuery query) {
        return PageResult.success(noticeService.getNoticePage(query));
    }

    @Operation(summary = "新增通知")
    @PostMapping
    @PreAuthorize("@ss.hasPerm('sys:notice:create')")
    @RepeatSubmit
    @Log(module = LogModuleEnum.NOTICE, value = ActionTypeEnum.INSERT)
    public Result<Long> createNotice(@Valid @RequestBody NoticeCreateForm form) {
        return Result.success(noticeService.createNotice(form));
    }

    @Operation(summary = "修改通知")
    @PutMapping("/{noticeId}")
    @PreAuthorize("@ss.hasPerm('sys:notice:update')")
    @RepeatSubmit
    @Log(module = LogModuleEnum.NOTICE, value = ActionTypeEnum.UPDATE)
    public Result<Void> updateNotice(
            @PathVariable @Positive Long noticeId,
            @Valid @RequestBody NoticeUpdateForm form
    ) {
        noticeService.updateNotice(noticeId, form);
        return Result.success();
    }

    @Operation(summary = "删除通知")
    @DeleteMapping
    @PreAuthorize("@ss.hasPerm('sys:notice:delete')")
    @Log(module = LogModuleEnum.NOTICE, value = ActionTypeEnum.DELETE)
    public Result<Void> deleteNotices(@RequestParam List<@Positive Long> ids) {
        noticeService.deleteNotices(ids);
        return Result.success();
    }
}
```

## 9. 验证

- 编译并运行模块相关测试。

- 至少覆盖：合法创建、校验失败、资源不存在、无权限、更新时 Path ID 生效、分页上限、事务回滚。

- 外部数据库或 Redis 不是测试目标时使用 Testcontainers、测试替身或专用 test profile，不依赖开发环境现存数据。

- 使用 ArchUnit 检查 `common` 不反向依赖 framework/业务域，业务域之间不存在 Entity 泄漏和循环依赖。

