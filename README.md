# izi-timepicker

## Install:
@inowave/izi-timepicker@git+https://repo.izibook.ru/scm/wvm/timepicker-component.git

## Config:
| Name 				  	| Type 					      | Default 			  	| Description
| ---- 				  	| :--: 					      | :-----: 			  	| ---------------
| value 		  		| `any`				    		| **null** 		  		| Реактивное значение
| maxHour 				| `Number`				    | **24** 				    | Максимальное занчение временного диапозона
| inclusive 			| `Boolean`	          | **false** 			  | Включить maxHour как завершающее значение времени
| underline 			| `Boolean`	          | **false** 			  | Включить у элементов нижнее подчёркивание
| prefix 				  | `String`	          | **''** 				    | Строковое значение перед временем
| disabled 				| `Boolean`	          | **false** 				| Заблокировать поле ввода
| width 					| `[String, Number]`	| **150** 			    | Ширина инпута
| height 			  	| `[String, Number]`	| **48** 			      | Высота инпута
| margin 			  	| `[String, Number]`	| **0** 					  | Внешнее смещение

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
