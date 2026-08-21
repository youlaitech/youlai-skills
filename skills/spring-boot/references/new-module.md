# 添加新模块 — 完整教程

以添加"通知公告"模块为例。

## Step 1: 创建实体

```java
// system/model/entity/SysNotice.java
@Data
@EqualsAndHashCode(callSuper = true)
@TableName("sys_notice")
@Schema(description = "通知公告")
public class SysNotice extends BaseEntity {

    /** 标题 */
    @Schema(description = "标题")
    private String title;

    /** 内容 */
    @Schema(description = "内容")
    private String content;

    /**
     * 通知类型.
     * <p>取值：{@code 1} 通知，{@code 2} 公告</p>
     */
    @Schema(description = "通知类型(1:通知 2:公告)")
    private Integer type;

    /**
     * 发布状态.
     * <p>取值：{@code 1} 已发布，{@code 0} 未发布</p>
     */
    @Schema(description = "发布状态(1:已发布 0:未发布)")
    private Integer status;
}
```

## Step 2: 创建 Mapper

```java
// system/mapper/SysNoticeMapper.java
@Mapper
public interface SysNoticeMapper extends BaseMapper<SysNotice> {
}
```

## Step 3: 创建 DTO

```java
// system/model/form/NoticeForm.java
@Data
@Schema(description = "通知公告表单")
public class NoticeForm {

    @NotBlank
    @Schema(description = "标题")
    private String title;

    @Schema(description = "内容")
    private String content;

    @Schema(description = "通知类型(1:通知 2:公告)")
    private Integer type;
}

// system/model/vo/NoticePageVO.java
@Data
@Schema(description = "通知公告分页对象")
public class NoticePageVO {

    @Schema(description = "ID")
    private Long id;

    @Schema(description = "标题")
    private String title;

    @Schema(description = "通知类型(1:通知 2:公告)")
    private Integer type;

    @Schema(description = "发布状态(1:已发布 0:未发布)")
    private Integer status;

    @Schema(description = "创建时间")
    private LocalDateTime createTime;
}

// system/model/query/NoticePageQuery.java
@Data
@EqualsAndHashCode(callSuper = true)
@Schema(description = "通知公告分页查询")
public class NoticePageQuery extends BaseQuery {

    @Schema(description = "关键字")
    private String keywords;

    @Schema(description = "通知类型(1:通知 2:公告)")
    private Integer type;
}
```

## Step 4: 创建 Converter

```java
// system/converter/NoticeConverter.java
@Mapper(componentModel = "spring")
public interface NoticeConverter {

    NoticePageVO toVO(SysNotice entity);

    List<NoticePageVO> toVOList(List<SysNotice> entities);

    SysNotice toEntity(NoticeForm form);
}
```

## Step 5: 创建 Service

```java
// system/service/NoticeService.java
public interface NoticeService {

    PageResult<NoticePageVO> getNoticePage(NoticePageQuery query);

    boolean saveNotice(NoticeForm form);

    void removeByIds(String ids);
}

// system/service/impl/NoticeServiceImpl.java
@Service
@RequiredArgsConstructor
public class NoticeServiceImpl implements NoticeService {

    private final SysNoticeMapper noticeMapper;
    private final NoticeConverter noticeConverter;

    @Override
    public PageResult<NoticePageVO> getNoticePage(NoticePageQuery query) {
        Page<SysNotice> page = new Page<>(query.getPageNum(), query.getPageSize());
        LambdaQueryWrapper<SysNotice> wrapper = Wrappers.lambdaQuery();
        wrapper.like(StrUtil.isNotBlank(query.getKeywords()), SysNotice::getTitle, query.getKeywords())
               .eq(query.getType() != null, SysNotice::getType, query.getType())
               .orderByDesc(SysNotice::getCreateTime);
        Page<SysNotice> result = noticeMapper.selectPage(page, wrapper);
        return PageResult.success(result.convert(noticeConverter::toVO));
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean saveNotice(NoticeForm form) {
        SysNotice notice = noticeConverter.toEntity(form);
        return noticeMapper.insertOrUpdate(notice);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void removeByIds(String ids) {
        List<Long> idList = Arrays.stream(ids.split(","))
            .map(Long::parseLong).collect(Collectors.toList());
        noticeMapper.deleteByIds(idList);
    }
}
```

## Step 6: 创建 Controller

```java
// system/controller/NoticeController.java
@Tag(name = "08.通知公告")
@RestController
@RequestMapping("/api/v1/notices")
@RequiredArgsConstructor
public class NoticeController {

    private final NoticeService noticeService;

    @Operation(summary = "通知分页列表")
    @GetMapping
    @Log(module = LogModuleEnum.NOTICE, value = ActionTypeEnum.LIST)
    public PageResult<NoticePageVO> getNoticePage(NoticePageQuery query) {
        return noticeService.getNoticePage(query);
    }

    @Operation(summary = "新增通知")
    @PostMapping
    @PreAuthorize("@ss.hasPerm('sys:notice:create')")
    @RepeatSubmit
    @Log(module = LogModuleEnum.NOTICE, value = ActionTypeEnum.INSERT)
    public Result<?> saveNotice(@Valid @RequestBody NoticeForm form) {
        return Result.judge(noticeService.saveNotice(form));
    }

    @Operation(summary = "删除通知")
    @DeleteMapping("/{ids}")
    @PreAuthorize("@ss.hasPerm('sys:notice:delete')")
    @Log(module = LogModuleEnum.NOTICE, value = ActionTypeEnum.DELETE)
    public Result<Void> removeByIds(@PathVariable String ids) {
        noticeService.removeByIds(ids);
        return Result.success();
    }
}
```

## 模块文件清单

```
system/
├── controller/NoticeController.java
├── converter/NoticeConverter.java
├── mapper/SysNoticeMapper.java
├── model/
│   ├── entity/SysNotice.java
│   ├── form/NoticeForm.java
│   ├── query/NoticePageQuery.java
│   └── vo/NoticePageVO.java
└── service/
    ├── NoticeService.java
    └── impl/NoticeServiceImpl.java
```
