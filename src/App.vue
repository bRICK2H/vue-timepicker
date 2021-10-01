<template>
	<div class="v-input-reference v-input-time-container"
		:class="[setClassFocusContainer, setClassDisabledContainer, inputRef]"
		:style="setStyleContainer"
		:tabindex="tabindex"
		:ref="inputRefContainer"
		@focus="focusContainer"
	>
		
		<div class="v-input-reference v-input-time-box"
			:class="inputRef"
		>
			<span v-show="prefix" 
				class="v-input-prefix"
			>
				{{ prefix }}
			</span>

			<input v-for="(value, i) of arrayValue"
				class="v-input-reference v-input-time"
				type="text"
				:class="[setClassActiveInput, inputRef, seClassUnderline]"
				:style="{ width: `${inputOptions[i].maxLen * 12}px` }"
				:key="i"
				:value="value"
				:ref="inputRef"
				:disabled="disabled"
				@blur="blur"
				@keydown="keydown($event, i)"
				@keypress="keypress($event)"
				@input="input($event, i)"
				@click="click($event, i)"
				@select="select(i)"
			>
		</div>

		<img src="./assets/img/clock.png" alt="clock">

	</div>
</template>

<script>
export default {
	name: 'VTimePicker',
	props: {
		value: null,
		maxHour: {
			type: Number,
			default: 24
		},
		inclusive: {
			type: Boolean,
			default: false
		},
		prefix: {
			type: String,
			default: ''
		},
		underline: {
			type: Boolean,
			default: false
		},
		disabled: {
			type: Boolean,
			default: false
		},
		width: {
			type: [String, Number],
			default: 150
		},
		height: {
			type: [String, Number],
			defautl: 48
		},
		margin: {
			type: [String, Number],
			defaut: 0
		}
	},
	data: () => ({
		inputRef: '',
		inputRefContainer: '',
		inputOptions: [
			{ maxLen: null, maxVal: null, minVal: 0 },
			{ maxLen: 2, maxVal: 60, minVal: 0 }
		],
		sideCoords: {
			pos: 0,
			start: 0,
			end: 0
		},
		typeValue: 'string',
		tabindex: 0,
		currIndex: 0,
		isInputCompleted: false,
		isFocus: false
	}),
	computed: {
		arrayValue() {
			let data = []

			if (typeof this.value === 'number') {
				this.typeValue = 'number'
				let hours = String(Math.floor(this.value / 60))
				let minutes = String(this.value - (+hours * 60))

				data = [hours, minutes]
			} else {
				data = this.value.split(':')
			}
			
			let [hours, minutes] = data
			const { maxVal } = this.inputOptions[0]

			hours = this.transformValue(hours)
			minutes = this.transformValue(minutes)
	
			if ((+hours * 60 + +minutes >= maxVal * 60 && !this.inclusive) || (+hours * 60 + +minutes > maxVal * 60 && this.inclusive)) {
				const modifyHours = this.getChangedValue(hours, 0)
				const modifyMinutes = this.getChangedValue(minutes, 1)

				return [modifyHours, modifyMinutes]
			}

			return [hours, minutes]
		},
		setClassActiveInput() {
			if (!this.underline) return null
			if (this.isFocus) {
				return this.currIndex === 0
					? 'v-input-time--f-active'
					: 'v-input-time--s-active'
			} else {
				return null
			}
		},
		setClassFocusContainer() {
			return this.isFocus ? 'v-input-time-container--focus' : null
		},
		setClassDisabledContainer() {
			return this.disabled ? 'v-input-time-container--disabled' : null
		},
		setStyleContainer() {
			return {
				maxWidth: `${this.width}px`,
				height: `${this.height}px`,
				margin: String(this.margin).split(' ').map(p => `${p}px`).join(' ')
			}
		},
		seClassUnderline() {
			return this.underline ? 'v-input-time--underline' : 'v-input-time--hideline'
		}
	},
	methods: {
		focusContainer() {
			this.isFocus = true

			if (!this.isInputCompleted) {
				this.writeDownCoords(0, 0, 1)
				this.$refs[this.inputRef][0].select()
			}
		},

		blur() {
			this.isInputCompleted = false
		},

		input(e, index) {
			const { target, data } = e
			const start = target.selectionStart
			const end = target.selectionEnd

			target.value = this.getChangedValue(target.value, index)

			this.switch(target, data, index, { start, end })
			this.save(index, target.value)
		},

		select(index) {
			const inputRef = this.$refs[this.inputRef]
			const { pos, start, end } = this.sideCoords
			this.currIndex = index

			if (inputRef.length) {
				inputRef[pos].setSelectionRange(start, end)
			}
			
		},

		click(e, index) {
			const { target } = e
			const { maxLen } = this.inputOptions[index]
			const start = target.selectionStart

			start === maxLen
				? this.writeDownCoords(index, start - 1, start)
				: this.writeDownCoords(index, start, start + 1)
			
			target.select()
		},

		keypress(e) {
			const { key } = e
			if (/[^\d]/.test(key)) e.preventDefault()
		},

		keydown(e, index) {
			const { target, code } = e
			const { selectionStart: start } = target
			const switchable = ['ArrowLeft', 'ArrowRight', 'Tab']
			const editable = ['ArrowUp', 'ArrowDown']
			const deletable = ['Delete', 'Backspace']

			// Переключение между инпутами и значениями
			if (switchable.includes(code) || deletable.includes(code)) {
				if (!this.isInputCompleted) {
					e.preventDefault()
				}

				this.switch(target, code, index)
			}

			// Пересчитать время при использовании стрелок
			if (editable.includes(code)) {
				e.preventDefault()
				this.edit(target, code, index)
			}

			// Удаление значений
			if (deletable.includes(code)) {
				this.delete(target, start, index)
			}
		},

		// Переключение между инпутами и значениями
		switch(target, code, index, options = {}) {
			const numbers = new Array(10).fill().map((c, i) => `${i}`)
			const inputRef = this.$refs[this.inputRef],
					start = target.selectionStart,
					end = target.selectionEnd,
					{ maxLen } = this.inputOptions[index]


			switch (code) {
				// Выделение значений в инпуте
				case numbers.find(c => c === code): {
					const { start, end } = options

					if (!index && end === maxLen) {
						this.writeDownCoords(1, 0, 1)
					} else {
						if (index && start === end) {
							this.writeDownCoords(index, 1, 2)
						} else {
							this.writeDownCoords(index, start, end + 1)
						}
					}

					inputRef[this.sideCoords.pos].select()

				}
					break

				case 'Backspace':
				case 'ArrowLeft': {
					if (start !== 0) {
						this.writeDownCoords(index, start - 1, end - 1)
						inputRef[index].select()
					} else {
						if (index) {
							inputRef[0].select()
							this.writeDownCoords(0, maxLen - 1, maxLen)
						} else {
							this.writeDownCoords(index, 0, 1)
							inputRef[index].select()
						}
					}
				}
					break

				case 'Tab':
				case 'Delete':	
				case 'ArrowRight': {
					if (code === 'Tab' && !!index) {
						if (!!index) {
							this.isInputCompleted = true

							if (end === maxLen) {
								this.isFocus = false
							}
						}
					}
					
					if (end !== maxLen) {
						this.writeDownCoords(index, start + 1, end + 1)
						inputRef[index].select()
					} else {
						if (!index) {
							this.writeDownCoords(1, 0, 1)
							inputRef[1].select()
						} else {
							this.writeDownCoords(index, maxLen - 1, maxLen)
							inputRef[index].select()
						}
					}
				}
					break
			}
		},

		// Удаление значений
		delete(target, start, index) {
			const arrValue = target.value.split('')

			if (!target.value) {
				target.value = new Array(maxLen).fill('0').join('')
			} else {
				arrValue.splice(start, 1, '0')
				target.value = arrValue.join('')
			}

			this.save(index, target.value)
		},

		// Изменить значение
		edit(target, code, index) {
			const { maxLen } = this.inputOptions[index]
			
			this.writeDownCoords(index, maxLen - 1, maxLen)
			target.select()
			
			switch (code) {
				case 'ArrowUp': {
					this.recalculate(true, index)
				}
					break
			
				case 'ArrowDown': {
					this.recalculate(false, index)
				}
					break
			
			}
		},

		// Обновить значение
		getChangedValue(value, index) {
			let minutes = null
			const arrValue = value.split('')

			if (this.typeValue === 'number') {
				let hours = String(Math.floor(this.value / 60))
				minutes = String(this.value - (+hours * 60))
			} else {
				const [, min] = this.value.split(':')
				minutes = min
			}

			const { maxVal } = this.inputOptions[index]
			const maxOffset = this.inclusive && !index && !+minutes ? maxVal + 1 : maxVal

			if (+value >= maxOffset) {
				arrValue.splice(arrValue.length - 1, 1)
				arrValue.unshift('0')
			}

			return arrValue.join('')
		},

		// Сохранить значение
		save(index, value) {
			this.arrayValue.splice(index, 1, value)
			
			if (this.typeValue === 'number') {
				const [hours, minutes] = this.arrayValue
				this.$emit('input', (+hours * 60) + +minutes)
			} else {
				this.$emit('input', this.arrayValue.join(':'))
			}
		},

		// Пересчитать значения
		recalculate(isCount, index) {
			const { maxVal, minVal } = this.inputOptions[index]
			let [hour, minutes] = this.arrayValue
			let counter = this.arrayValue[index]
			counter = isCount
				? ++counter
				: --counter

			const formatValue = value => {
				if (index) {
					const {
						maxVal: maxHour,
						minVal: minHour
					} = this.inputOptions[0]

					if (value === -1) {
						hour = +hour === 0 ? maxHour - 1 : --hour
					} else if (value === 60) {
						const maxOffst = this.inclusive ? maxHour : maxHour - 1 
						hour = +hour === maxOffst ? minHour : ++hour
					} else if (this.inclusive) {
						if (value === 1 && +hour === maxHour) {
							hour = minHour
						} else if (value === 0 && +hour === 0) {
							hour = maxHour
						}
					}

					this.arrayValue.splice(0, 1, hour)
				}
					
					value = value === maxVal || value === minVal
						? !index && !+minutes && this.inclusive
							? maxVal
							: minVal
						: value

					value = value === maxVal + 1 && !index && !+minutes && this.inclusive ? minVal + 1 : value

					value = value === minVal - 1
						? maxVal - 1
						: value

				return value
				
			}
			
			this.save(index, formatValue(counter))
		},

		// Преобразование значения с нулём
		transformValue(val) {
			val = +val
			return val < 10 && val >= 0 ? `0${val}` : `${val}`
		},

		// Перезаписать координаты курсора при перемещении
		writeDownCoords(pos, start, end) {
			this.sideCoords.pos = pos
			this.sideCoords.start = start,
			this.sideCoords.end = end
		},

	},
	watch: {
		isFocus(focus) {
			if (!focus) this.$emit('blur')
		}
	},
	created() {
		this.inputRef = `input-time-ref:${String(Math.random()).slice(2, 10)}`
		this.inputRefContainer = `input-container:${String(Math.random()).slice(2, 10)}`
		this.inputOptions[0].maxLen = +String(this.maxHour).length
		this.inputOptions[0].maxVal = this.maxHour
	},
	mounted() {
		this.writeDownCoords(0, 0, 1)

		document.addEventListener('click', e => {
			this.isFocus = e.target.classList.contains(this.inputRef)
		})
	}
}
</script>

<style lang="scss">
	.v-input-time-container {
		width: 100%;
		height: 48px;
		display: flex;
		justify-content: space-around;
		align-items: center;
		user-select: none;
		background: #fff;
		border: 2px solid #eeedf7;
		border-radius: 12px;

		&--focus {
			border: 2px solid #cfcde5;
		}
		&--disabled {
			opacity: 0.6;
		}
	}
	.v-input-time-box {
		position: relative;

		&::after {
			content: ':';
			position: absolute;
			top: calc(50% - 1px);
			right: 30px;
			transform: translateY(-50%);
			font-size: 16px;
			font-weight: 900;
			color: #76768C;
		}
	}
	.v-input-time {
		width: 0;
		min-width: 30px;
		text-align: center;
		font-size: 14px;
		font-weight: 500;
		outline: none;
		border: none;
		color: #76768C;

		&--hideline {
			border-bottom: none;
		}
		&--underline {
			padding-bottom: 2px;
			border-bottom: 2px solid #eeedf7;
			transition: .1s;
		}

		&:focus {
			color: #76768C;
		}
		&:first-of-type {
			margin-right: 2px;
		}
		&:last-of-type {
			margin-left: 2px;
		}
		&--f-active {
			&:first-of-type {
				border-bottom: 2px solid #cfcde5;
			}
		}
		&--s-active {
			&:last-of-type {
				border-bottom: 2px solid #cfcde5;
			}
		}
	}
	.v-input-prefix {
		color: #76768C;
	}
</style>