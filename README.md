# Mapster 使用說明

## 套件自動產生Model
### 安裝Mapster.Tool
``` PowerShell
dotnet new tool-manifest 
dotnet tool install Mapster.Tool
```

並在csproj檔案加入
手動:

``` XML
  <Target Name="Mapster">
    <Exec WorkingDirectory="$(ProjectDir)" Command="dotnet build" />
    <Exec WorkingDirectory="$(ProjectDir)" Command="dotnet tool restore" />
    <Exec WorkingDirectory="$(ProjectDir)" Command="dotnet mapster model -a &quot;$(TargetDir)$(ProjectName).dll&quot;" />
    <Exec WorkingDirectory="$(ProjectDir)" Command="dotnet mapster extension -a &quot;$(TargetDir)$(ProjectName).dll&quot;" />
    <Exec WorkingDirectory="$(ProjectDir)" Command="dotnet mapster mapper -a &quot;$(TargetDir)$(ProjectName).dll&quot;" />
  </Target>
```
``` powershell
dotnet msbuild -t:Mapster
```

自動:
``` XML
  <Target Name="Mapster" AfterTargets="AfterBuild">
    <Exec WorkingDirectory="$(ProjectDir)" Command="dotnet tool restore" />
    <Exec WorkingDirectory="$(ProjectDir)" Command="dotnet mapster model -a &quot;$(TargetDir)$(ProjectName).dll&quot;" />
    <Exec WorkingDirectory="$(ProjectDir)" Command="dotnet mapster extension -a &quot;$(TargetDir)$(ProjectName).dll&quot;" />
    <Exec WorkingDirectory="$(ProjectDir)" Command="dotnet mapster mapper -a &quot;$(TargetDir)$(ProjectName).dll&quot;" />
  </Target>
```

命令說明:
```
dotnet mapster model //生成Model
dotnet mapster extension //生成擴充方法
dotnet mapster mapper //生成從介面產生的映射方法
```
```
-a 設定執行的.dll檔案
-n 設定namespace
-o 設定輸出目錄
-r 輸出為record(.net5 up) 而不是class

```


產生對應的Dto(使用Attribute)
```
[AdaptTo("[name]Dto"),GenerateMapper]
public class Member
{
    public string Account { get; set; }

    public string Password { get; set; }

    public string Email { get; set; }

}
```
產生對應的Dto(使用Fluent API)
``` csharp
public class MemberRegister: ICodeGenerationRegister
{
	public void Register(CodeGenerationConfig config)
	{
	       config.AdaptTo("[name]Dto").ForType<Member>(cfg=>
	       {
	       		//設定欄位名稱或欄位型別
	       		cfg.Map(m => m.Account, "Name");   
        		cfg.Map(m => m.Password, typeof(Guid)); 
	       });

 	       config.GenerateMapper("[name]Mapper")
 	               .ForType<Member>();
	}
}

```

> :warning: 不建議在專案上使用自動產生Model，除非大家都有裝套件

>設定完之後build project會產生MemberDto以及MemberMapper擴充方法
但第一次build的時候MemberMapper裡面會沒有內容
這是因為第一次build的時候dto並不存在
所以需要再rebuild一次
是rebuild不是build，第一次build完後沒有修改內容，第二次build會使用第一次build的內容，導致MemberMapper還是空的

## 開始使用
### web api 安裝套件
``` PowerShell
Install-Package Mapster
Install-Package Mapster.DependencyInjection
```

### Dependency Injection
startup or Program
``` csharp
    var config = TypeAdapterConfig.GlobalSettings;
    services.AddSingleton(config);
    services.AddScoped<IMapper, ServiceMapper>();
```

### 映射使用說明:
簡單映射:跟autoMapper依樣，可不用設定config
> :warning: 建議設定config

```csharp
    注入IMapper
    private readonly IMapper _mapper;
    public class(Imapper mapper) => _mapper = mapper
	
    var memberDto = _mapper.Map<MemberDto>(member);
```
擴充方法映射，有使用GenerateMapper會產生擴充方法
```csharp
var member = new Member
{
	Account = "Tom",
	Password = "123",
	Email = "email"
}
var memberDto = member.AdaptToDto();
```
自訂映射
```csharp
public class MemberRegister : IRegister
{
	public void Register(TypeAdapterConfig config)
	{
		config.NewConfig<Member, MemberDto>();
			 .Map(m => m.Email, o => o.Password)
	    //雙向映射
            .TwoWays();
          
	}
}
```



## 參考:
[Mapster](https://github.com/MapsterMapper/Mapster)

