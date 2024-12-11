#### Blue print
###### Pure function
side effect가 없는 함수. side effect란 해당 함수를 실행하고 난 뒤 어떠한 현상이 일어나는 것.
오직 반환값만이 있음.
노드에 exec 핀을 꽂을 필요가 없는 함수.

**디테일 패널에 Pure 체크박스 활성화 시 Pure 함수로 작동**
###### self node
this reference
#### C++
###### UPROPERTY
변수를 에디터에서 수정 가능하게 만들어줌.
```cpp
UPROPERTY(EditAnywhere, Category = "Category")
int32 variable;
```

cf) UPROPERTY()의 변수를 에디터에서 수정하고 라이브 코딩으로 빌드를 하면 당장에는 적용되지만 에디터를 나가면 적용이 안되는 현상이 일어남. 이럴때는 IDE에서 빌드를 다시 하고 에디터를 켜면 적용됨.
