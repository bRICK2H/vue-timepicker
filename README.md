# timepicker

## Config:
| Name 				  	| Type 					      | Default 			  	| Description
| ---- 				  	| :--: 					      | :-----: 			  	| ---------------
| value 		  		| `any`				    		| **null** 		  		| Реактивное значение
| stopTime        | `Object`            | **null**          | Смотреть в примечании 'stopTime'
| maxHour 				| `Number`				    | **24** 				    | Максимальное занчение временного диапозона
| inclusive 			| `Boolean`	          | **false** 			  | Включить maxHour как завершающее значение времени
| underline 			| `Boolean`	          | **false** 			  | Включить у элементов нижнее подчёркивание
| side 			      | `String`	          | **left** 			    | Позиция времени (left of right)
| prefix 				  | `String`	          | **''** 				    | Строковое значение перед временем
| icons 				  | `String`	          | **'clock'** 			| Выбрать иконку: clock | hourglass
| disabled 				| `Boolean`	          | **false** 				| Заблокировать поле ввода
| width 					| `[String, Number]`	| **150** 			    | Ширина инпута
| maxWidth 				| `[String, Number]`	| **150** 			    | Максимальная ширина инпута
| height 			  	| `[String, Number]`	| **48** 			      | Высота инпута
| margin 			  	| `[String, Number]`	| **0** 					  | Внешнее смещение

## Node:
```javascript
  /**
   * @type 'while' (пока)
   * - Изменяется, пока вычисляемое значение не будет <= stopTime.value
   */
  stopTime: {
    type: 'while',
    value: 24 * 60 - end
  }

  /**
   * @type 'until' (до)
   * - Изменяется до момента, пока вычисляемое значение не достигнет stopTime.value
   */
  stopTime: {
    type: 'until',
    value: 60 * 15
  }
```

## Example:
```html
<VTimePicker
  prefix="c"
  :maxHour="24"
  :width="176"
  :inclusive="true"
  margin="0 8 0 0"
  v-model="time"
/>
