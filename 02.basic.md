# 위젯클래스의 정의
위젯은 먼저 클래스로 작성되어야 하며 클래스당 1개의 js 파일이 작성되어야 한다. 이 js파일의 이름은  widget의 full name(naspace+alias)과 일치하여야 하는 이름 규칙을 가진다. 예를들어 아래와 같이 Button클래스는 xwing.widget.button.js에 저장된다.

* code:title=xwing.widget.button.js
```
Class.define({
	Button : { //Widget 클래스의 이름
		alias : 'button', // Widget의 name
		namespace : 'xwing.widget', //xwing.widget.Button 으로 클래스가 생성됨
		extend : xwing.widget.DataBindable, //상위 클래스 지정
		Button : function(json){ //위젯의 생성자
			this._init(json); //디폴트 코드
		},
		statics : { //Class의 함수. 즉,xwing.widget.Button.method() 와 같이 접근할 수 있는 함수
			create : function(json){ //instance생성의  factory 함수
				return new xwing.widget.Button(json); //디폴트 코드
			},
			.... (기타함수를 오버라이딩 하거나 추가함)
		},		
		prototypes : { // instance함수. 즉 this.method()로 접근할 수 있는 함수
			createPart : function(){ //위젯의 기본 DOM골격의 생성을 담당하는 callback 함수

			},	
			render : function(){ //위젯의 기본 골격에 스타일링을 담당하는 callback함수
			},
			.... (기타함수를 오버라이딩 하거나 추가함)
		}	
	}
});
```

* Class.define()함수가 클래스를 정의한 후 alias를 key로 사용해서 Class._map에 저장한다. 저장된 Class를 다음과 같은 방식으로 얻어내고 사용하게 된다.
```
var widgetClass = Class.getClass(alias);
var widgetInstance = new widgetClass(option);
```

# 위젯의 종류와 상속
Xwing의  위젯은 상속 구조를 가지고 있으며, 위젯은 종류에 따라 상속을 받는 상위 클래스가 달라진다.

* 최상위 클래스 - xwing.widget.Widget
    * Xwing의 모든 위젯 클래스는 xwing.widget.Widget을 상속 받는다. Widget클래스는 Xwing에서 위젯이 가져야할 기본 속성과 동작을 사전에 구현 해놓았기 때문에 이 클래스를 상속 받지 않으면 위젯으로서 기능을 할 수 없다.    
    이 클래스를 상속 받음으로서 위젯클래스들은 위젯이 되기 위한 동작을 따로 구현할 필가 없다.

* Data binding 되는 것 - xwing.widget.Dababindable
    * data binding을 위한 기본 기능을 정의해 놓은 클래스가 Databindable이다. Xwing에서 data binding은 xwing.widget.Dataset과의 연동을 의미한다. 많은 위젯이 data binding을 필요로 하는데 그러한 위젯은 Databindable클래스를 상속받아 구현한다.    
    모든 Form요소와 Grid,Tree등의 고급위젯을 포함하여 대부분의 것이 여기에 포함된다.

* 다름 위젯을 담을 수 있는 것 - xwing.widget.Panel
    * 하위 위젯들을 담는 역할을 하는 container 위젯을 구현할 때 사용하는 위젯이다. 이것은 상속을 위한 상위 클래스가 아니라, 그 자체로 xwing사용자가 직접 사용할 수 있는 위젯이다.    
    Panel의 기본 기능을 넘어서는 또 다른 container 위젯을 구현할 때 상속 받아 사용한다.   
    Panel, Group, Tabpage등 다른 위젯을 그 안에 배치 시킬수 있는능력을 가진 것들이 여기에 포함된다.

* 서브위젯을 가지는 것
    * Tab,Grid등이 여기 속하며 각각 Tabpage, Gridcolumn등을 서브 위젯을 가지고 있으며 이 서브 위젯은 단독으로 사용되지 않는다. 보통 Tab이나 Grid와 같은 위젯은 서브위젯을 담기 때문에 xwing.widget.Panel을 상속받아  구현되었다고 생각할 수 있으나, Panel의 기능을 변형하는 것 만으로 서브위젯과의 밀접한 연결을 구현하기 어렵기 때문에 xwing.widget.Widget을 상속받아 구현되었다. 서브 위젯인 Tabpage,Gridcoumn등은 필요에 따라 상속받는 상위 클래스가 다르다.  
     주의할 점은 위젯과 그 서브위젯은 상속과는 무관한 개념이다

* 정리하여 위젯의 상속 구조를 트리로 표시하면 아래와 같다.
    * xwing.widget.Widget
        * xwing.widget.Databindable
            * xwing.widget.Button
            * xwing.widget.Tree
            * (그외 많은 위젯들)
        * xwing.widget.Panel
            * xwing.widget.Page
            * xwing.widget.Tabpage
        * xwing.widget.Tab
        * xwing.widget.Grid
        * xwing.widget.Gridcolumn


# 위젯의 속성정의
위젯은 많은 속성을 가지고 있다. width,height,font-size,color등 UI적 요소를 위한 것도 있으며, editable, checkable등 기능적인 요소 뿐아니라, binddataset, value와 같은 위젯의 데이터를 위한 속성도 있다.
이 모든 속성은 모두 model js파일에 정의된다. model js파일은  이름규칙을 가지는데 예를 들어, xwing.widget.button.js가 위젯의 파일이라면 xwing.widget.button.model.js라는 이름의 파일에 정의되며 형식은 아래에 예시한 바와 같다.

* xwing.widget.model.js
```
xwing.widget.Button._model=
{
	attributes:{
		height :{
			defaultValue:'22'
		},
		fontfamily:{
			defaultValue:'NanumGothicExtraBold'
		},
		fontweight : {
			defaultValue:null
		},
		fontcolor:{
			defaultValue:null
		},
		bgcolor:{
			defaultValue:null
		},
		icon : {
			type : 'string',
			defaultValue : null,
			valueDomain : null,
			control : 'image'
		},
		mask : null
	},
	events : ['click', 'mouseover', 'mouseout',  'focusin', 'focusout']
};
```

예시와 같이 속성은 xwing.widget.Button._model이라는 클래스 변수에 json객체를 할당해 주는 방식으로 정의한다.    
기본적으로 attributes와 events 를 정의하는데 events와 관련해서는 아래에서 설명하고 attributes만을 설명하기로 한다    
attributes속성 하위에는 위젯이 사용하는 속성을 나열하는데, 그 형식은 다음과 같다.
```
width:{
			type:'number',                  //속성의 type을 number|string|boolean으로 정의
			defaultValue:'100',             //속성의 기본값을 정의
			valueDomain:'(-|)+[0-9]+(px|)', //속성이 가질수 있는 값의 리스트를 정규식으로 정의. 임의의 값이면 null
			category:'Bounds',              //속성의 Category, IDE의 property 편집창에서 사용됨
			control:'edit'                  //속성의 편집 Control, IDE의 property 편집창에서 사용됨
		}
```


* 속성의 상속
    * 위젯뿐아니라, 위젯의 상위 클래스들도 model을 정의 하고 있다. 따라서, xwing.widget.Widget클래스에서 정의하는 속성이 xwing.widget.model.js에 정의되어 있는데 위젯클래스가 xwing.widget.Widget클래스를 상속 받고 있다면 그에 따른 속성도 자동으로 상속된다.

* 속성의 재정의
    * 상속 받은 속성을 재정의 하고 싶다면 다음과 같이 해당 속성의 필요부분만을 다시 기술한다.
```
width :{
			defaultValue:'22'
		},
```

* 속성의 상속 포기
    * 하위위젯이 상위위젯의 어떤 속성은 필요하지 않아 사용하지 않는다고 한다면 다음과 같이 기술한다.
```
mask : null
```


# 위젯의 사용자 이벤트 정의
위젯은 위젯의 고유의 이벤트들 정의 할 수 있다. Tree를 예로 들자면 특정 노드를 선택했을 때, 폴더를 펼쳤을 때, 선택이 변경되었을 때 등등...    
각각의 이벤트들은 이벤트이름을 가지고 있으며, 각 위젯은 이를 model js파일에 선언해야한다.  xwing.widget.Widget클래스는 모델에 모든 위젯에서 사용가능한 이벤트의 리스트를 기술해 놓았는데, 이를 예시하여 위젯이 이벤트의 종류를 기술하는 방법은 아래와 같다.
* xwing.widget.Widget.model.js
```
xwing.widget.Widget._model={
	attributes:{
	...
	},
	events:['click','mouseover','mouseout', 'focusin', 'focusout']
}
```

이렇게 나열된 이벤트의 종류에 대해서 각 위젯클래스는 row level의 DOM이벤트를 잡아서 조건에 맞을 경우 위젯 고유의 이벤트를 발생시킨다. Widget클래스는 위의 5가지 이벤트들에 대해서 이벤트처리를 구현해 놓았으며, 하위 위젯들은 이러한 이벤트에 대한 추가코드를 구현하지 않아도 된다.   

하지만, 하위 위젯에서 위의 5가지 이벤트를 사용하고 싶다면, event의 이름은 나열해야한다. 왜냐하면, event의 이름을 기술하지 않은면 해당 이벤트는 사용하지 않겠다는 의미로 간주되기 때문이다. 따라서, 공통의 이벤트이외에 추가 이벤트가 필요하다면 다음과 같이 나열해야한다.    

Tree의 경우를 예로 들면 다음과 같다.
```
events:['click', 'mouseover', 'mouseout',  'focusin', 'focusout', 'select','change','dblclick','scroll','scrolltop','scrollbottom','expand','collapse']
```

# 위젯의 스타일링
Xwing의 위젯은 그 기능과 모양을 구현하기 위해서 많은 html의 DOM요소를 사용하게 된다. 즉, html 태그를 사용해서 작성하게 되는데, html의 권고에 따라 DOM의 문서구조 자체와 스타일링을 분리하기 위해서 css를 사용하게 된다. Xwing 위젯을 작성할 때도 같은 방식을 사용하는데, 이를 위해서 위젯을 작성할 때 사용한 모든 하위 html요소는 class 아이디를 부여 받아야 한다. 이렇게 부여된 각각의 class 아이디에 css style을 지정한 css파일이  각  위젯마다  존재하는데 xwing.widget.button.js라면 xwing.button.css라는 파일명을 가져야 하는 이름 규칙을 가진다.

* xwing.widget.button.css
```
.xw-button-shell {
	...
}

.xw-button {
	...
}
.xw-button-icon{
	...
}
.xw-button-enabled {
	...
}

.xw-button-enabled:hover {
	...
}

.xw-button-enabled:active,.xw-button-click {
	...
}

.xw-button-disabled {
	...
}
```

대략 위와 같이 보이는데 보다시피 class 아이디를 부여하는 이름규칙이 존재한다. 그 이름 규칙의 첫번째는 모든 아이디가 xw-위젯이름 으로 시작한다는 점이고 그 외 상세한 규칙은 위젯에 대한 자세한 설명으로 미루도록 하겠다.

# 위젯의 등록
하나의 위젯이 작성되려면, 지금까지 설명한 내용에 따라서 다음의 3가지 파일이 필요하다.
* 위젯클래스파일 (xwing.widget.button.js)
* 위젯모델파일 (xwing.widget.button.model.js)
* 위젯CSS파일 (xwing.button.css)

그렇다면, 이렇게 생성된 파일을 어떻게 하여야 Xwing의 위젯으로 등록될 수 있는가를 설명할 필요가 있다.
Xwing은 page가 브라우저에 로딩될때 동작하는 javascript 코드가 있는데 그것은 xwing.js에 정의되어 있다. 여기에 다음 부분이 필요한 js파일과 css파일을  로딩하도록 등록하여야 하는 부분이다.
* xwing.js
```
,requiredJs : [ 
               .....,
               ,"/js/xwing.widget.button.js" ,"/js/xwing.widget.button.model.js"
               ....
]
,requiredCss : [ ...
               ,"/css/themes/navy/xwing.css"
               ]
```

위에서 보면 xwing.button.css를 등록한 부분은 보이지 않는다. 그것은 xwing.css에 다음과 같이 정의해야 하는 부분이다.
* xwing.css
```
....
@IMPORT url("xwing.button.css");
....
```


이제 위에서 작성된 xwing.widget.Button은 위젯으로 xwing에 등록되었으며 사용가능 해진다.

