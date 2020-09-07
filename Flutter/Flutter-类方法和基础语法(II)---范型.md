># 范型限制
```
/*
*  <T extends BaseStatefulWidget,K extends BaseBloc>: 限制传入的类型是 BaseStatefulWidget(或者是其子类) 和 BaseBloc(或者是其子类)
* TemplateBarState<T>： 表示继承自 TemplateBarState(或者是其子类)， <T> 表示限制传入的类型是 BaseStatefulWidget (或者是其子类)
*/
abstract class TemplateBlocBarState<T extends BaseStatefulWidget,
    K extends BaseBloc> extends TemplateBarState<T> {
  K kBloc;

  @override
  void initState() {
    super.initState();
    kBloc = absKBloc();
  }

  K absKBloc();
}



/* 
*  SettingPage 继承自：BaseStatefulWidget
*  UserBloc 继承自：AbsUserBloc， AbsUserBloc 继承自：BaseBloc
*/
class _SettingPageState extends TemplateBlocBarState<SettingPage, UserBloc> { 


}

```
