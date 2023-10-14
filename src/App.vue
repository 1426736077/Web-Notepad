<script>
import { computed, reactive, readonly, ref ,shallowReactive, shallowReadonly, watch, watchEffect} from 'vue';
export default{
	data(){
		return{
			tasks:[
				
			],
			visibility:'all',
			toggleAll:true,
			s:'个剩余项目'
		}
	},
	watch:{
		tasks:{
      		handler(newValue) {
        	if(this.tasks.every(task => task.completed === false)){
				this.toggleAll = false
			}else{
				this.toggleAll = true
			}
        	// 这里是数组变化时的操作
			localStorage.setItem('task',JSON.stringify(this.tasks))
      		},
      		deep: true,
    	}
	},
	methods:{
		allcheck(e){ 
			this.tasks.forEach(it=>{
				it.completed = e.target.checked
			})
		},
		onHashChange(){
			const hash = window.location.hash.replace(/#\/?/,'')  //清除hash值的前缀#/
			// console.log(hash);
			if(['all','active','completed'].includes(hash)){ //判断hash值是否包含其中一个指定的值
				// console.log(hash);
				this.visibility=hash  
			}else{
				// console.log(2);
				window.location.hash = ''
				this.visibility = 'all'
			}
			// console.log(this.visibility);
		},
		addTask(e){
			const task = e.target.value.trim()  //trim() 方法会从字符串的两端移除空白字符，并返回一个新的字符串
			if(!task){
				return
			}
			else{
				this.tasks.push({
					// resive:false,
					task,
					completed:false
				})
			}
			e.target.value = ''
		},
		deleteTask(task){
			this.tasks.splice(this.tasks.indexOf(task),1) //从tasks中得出task所在序列并从该位置处删除一个元素
			
		},
		clearTask(){
			this.tasks = this.tasks.filter(task=> !task.completed) //filter遍历数组虽然判断为true返回最终返回一个新数组
		},
	    async onRe(item){
		    await(item.revise = !item.revise)  //先通过异步函数通过await异步执行等待完成后让元素渲染后给元素聚焦处理 会自动添加revise值
			// console.log(this.$refs.reviseTask[this.tasks.indexOf(item)]);
			// console.log(item.task);
			this.$refs.reviseTask[this.tasks.indexOf(item)].value = item.task
			this.$refs.reviseTask[this.tasks.indexOf(item)].focus()
			// this.$refs.reviseTask[this.tasks.indexOf(item)].style.color = "red"
		},
		offRe(item,event){
			item.revise = !item.revise
			item.task = this.$refs.reviseTask[this.tasks.indexOf(item)].value
			if(!item.task){
				this.tasks.splice(this.tasks.indexOf(item), 1)
			}
			document.querySelector('.new-todo').focus()  //完成修改后重新聚焦到输入
			event.preventDefault();
		},
	},
	computed:{
		fileredTodos(){
			switch(this.visibility){
				case 'all': return this.tasks
				case 'active': return this.tasks.filter(task => !task.completed)
				case 'completed': return this.tasks.filter(task => task.completed)
			}
		},
		count(){
			return this.tasks.filter(task=>!task.completed).length
		}
	},
	mounted(){
		window.addEventListener('hashchange',this.onHashChange) //给窗口添加hashchange事件当 URL 的片段标识符更改时，将触发hashchange事件 
		this.onHashChange() //为了防止刷新后过滤失效在组件渲染时进行一次调用
		this.tasks=JSON.parse(localStorage.getItem('task'))||[]
	}
}
</script>

<template>
  <section class="todoapp">
			<header class="header">
				<h1>代办事项</h1>
				<input class="new-todo" placeholder="需 要 做 什 么 ?" autofocus @keyup.enter="addTask">  
                <!-- autofocu 默认聚焦 -->
			</header>
			<!-- This section should be hidden by default and shown when there are todos -->
			<section class="main">
				<input id="toggle-all" class="toggle-all" type="checkbox" @click="allcheck" v-model="toggleAll">
				<label for="toggle-all">Mark all as complete</label>
				<ul class="todo-list">
					<!-- These are here just to show the structure of the list items -->
					<!-- List items should get the class `editing` when editing and `completed` when marked as completed -->
					<li v-if="fileredTodos.length > 0" v-for="item in fileredTodos" :class="{'completed':item.completed,'editing':item.revise}">
						<div class="view">
							<input class="toggle" type="checkbox" v-model="item.completed">
							<label @dblclick="onRe(item)">{{ item.task }}</label>
							<!-- contenteditable是一个枚举属性表示元素是否可被用户编辑  通过$event传入事件对象-->
							<button class="destroy" @click="deleteTask(item)" autofocus></button>
						</div>
						<input ref="reviseTask" class="edit" @blur="offRe(item,$event)">
					</li>
				</ul>
			</section>
			<!-- This footer should be hidden by default and shown when there are todos -->
			<footer class="footer">
				<!-- This should be `0 items left` by default -->
				<span class="todo-count"><strong>{{count}}</strong> {{ s }}</span>
				<!-- Remove this if you don't implement routing -->
				<ul class="filters">
					<li>
						<a :class="{selected:visibility === 'all'}" href="#/all">全部</a>
					</li>
					<li>
						<a :class="{selected:visibility === 'active'}" href="#/active">未完成</a>
					</li>
					<li>
						<a :class="{selected:visibility === 'completed'}" href="#/completed">已完成</a>
					</li>
				</ul>
				<!-- Hidden if no completed items are left ↓ -->
				<button :class="{'clear-completed':toggleAll}" @click="clearTask">清空已完成任务</button>
			</footer>
		</section>
</template>

<style>
html,
body {
	margin: 0;
	padding: 0;
}

button {
	margin: 0;
	padding: 0;
	border: 0;
	background: none;
	font-size: 100%;
	vertical-align: baseline;
	font-family: inherit;
	font-weight: inherit;
	color: inherit;
	-webkit-appearance: none;
	appearance: none;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
}

body {
	font: 14px 'Helvetica Neue', Helvetica, Arial, sans-serif;
	line-height: 1.4em;
	background: #f5f5f5;
	color: #111111;
	min-width: 230px;
	max-width: 550px;
	margin: 0 auto;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	font-weight: 300;
}

.hidden {
	display: none;
}

.todoapp {
	background: #fff;
	margin: 130px 0 40px 0;
	position: relative;
	box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2),
	            0 25px 50px 0 rgba(0, 0, 0, 0.1);
}

.todoapp input::-webkit-input-placeholder {
	font-style: italic;
	font-weight: 400;
	color: rgba(0, 0, 0, 0.4);
}

.todoapp input::-moz-placeholder {
	font-style: italic;
	font-weight: 400;
	color: rgba(0, 0, 0, 0.4);
}

.todoapp input::input-placeholder {
	font-style: italic;
	font-weight: 400;
	color: rgba(0, 0, 0, 0.4);
}

.todoapp h1 {
	position: absolute;
	top: -140px;
	width: 100%;
	font-size: 80px;
	font-weight: 200;
	text-align: center;
	color: #b83f45;
	-webkit-text-rendering: optimizeLegibility;
	-moz-text-rendering: optimizeLegibility;
	text-rendering: optimizeLegibility;
}

.new-todo,
.edit {
	position: relative;
	margin: 0;
	width: 100%;
	font-size: 24px;
	font-family: inherit;
	font-weight: inherit;
	line-height: 1.4em;
	color: inherit;
	padding: 6px;
	border: 1px solid #999;
	box-shadow: inset 0 -1px 5px 0 rgba(0, 0, 0, 0.2);
	box-sizing: border-box;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
}

.new-todo {
	padding: 16px 16px 16px 60px;
	height: 65px;
	border: none;
	background: rgba(0, 0, 0, 0.003);
	box-shadow: inset 0 -2px 1px rgba(0,0,0,0.03);
}

.main {
	position: relative;
	z-index: 2;
	border-top: 1px solid #e6e6e6;
}

.toggle-all {
	width: 1px;
	height: 1px;
	border: none; /* Mobile Safari */
	opacity: 0;
	position: absolute;
	right: 100%;
	bottom: 100%;
}

.toggle-all + label {
	display: flex;
	align-items: center;
	justify-content: center;
	width: 45px;
	height: 65px;
	font-size: 0;
	position: absolute;
	top: -65px;
	left: -0;
}

.toggle-all + label:before {
	content: '❯';
	display: inline-block;
	font-size: 22px;
	color: #949494;
	padding: 10px 27px 10px 27px;
	-webkit-transform: rotate(90deg);
	transform: rotate(90deg);
}

.toggle-all:checked + label:before {
	color: #484848;
}

.todo-list {
	margin: 0;
	padding: 0;
	list-style: none;
}

.todo-list li {
	position: relative;
	font-size: 24px;
	border-bottom: 1px solid #ededed;
}

.todo-list li:last-child {
	border-bottom: none;
}

.todo-list li.editing {
	border-bottom: none;
	padding: 0;
}

.todo-list li.editing .edit {
	display: block;
	width: calc(100% - 43px);
	padding: 12px 16px;
	margin: 0 0 0 43px;
}

.todo-list li.editing .view {
	display: none;
}

.todo-list li .toggle {
	text-align: center;
	width: 40px;
	/* auto, since non-WebKit browsers doesn't support input styling */
	height: auto;
	position: absolute;
	top: 0;
	bottom: 0;
	margin: auto 0;
	border: none; /* Mobile Safari */
	-webkit-appearance: none;
	appearance: none;
}

.todo-list li .toggle {
	opacity: 0;
}

.todo-list li .toggle + label {
	/*
		Firefox requires `#` to be escaped - https://bugzilla.mozilla.org/show_bug.cgi?id=922433
		IE and Edge requires *everything* to be escaped to render, so we do that instead of just the `#` - https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/7157459/
	*/
	background-image: url('data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A//www.w3.org/2000/svg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%23949494%22%20stroke-width%3D%223%22/%3E%3C/svg%3E');
	background-repeat: no-repeat;
	background-position: center left;
}

.todo-list li .toggle:checked + label {
	background-image: url('data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%2359A193%22%20stroke-width%3D%223%22%2F%3E%3Cpath%20fill%3D%22%233EA390%22%20d%3D%22M72%2025L42%2071%2027%2056l-4%204%2020%2020%2034-52z%22%2F%3E%3C%2Fsvg%3E');
}

.todo-list li label {
	word-break: break-all;
	padding: 15px 15px 15px 60px;
	display: block;
	line-height: 1.2;
	transition: color 0.4s;
	font-weight: 400;
	color: #484848;
}

.todo-list li.completed label {
	color: #949494;
	text-decoration: line-through;
}

.todo-list li .destroy {
	display: none;
	position: absolute;
	top: 0;
	right: 10px;
	bottom: 0;
	width: 40px;
	height: 40px;
	margin: auto 0;
	font-size: 30px;
	color: #949494;
	transition: color 0.2s ease-out;
}

.todo-list li .destroy:hover,
.todo-list li .destroy:focus {
	color: #C18585;
}

.todo-list li .destroy:after {
	content: '×';
	display: block;
	height: 100%;
	line-height: 1.1;
}

.todo-list li:hover .destroy {
	display: block;
}

.todo-list li .edit {
	display: none;
}

.todo-list li.editing:last-child {
	margin-bottom: -1px;
}

.footer {
	padding: 10px 15px;
	height: 20px;
	text-align: center;
	font-size: 15px;
	border-top: 1px solid #e6e6e6;
}

.footer:before {
	content: '';
	position: absolute;
	right: 0;
	bottom: 0;
	left: 0;
	height: 50px;
	overflow: hidden;
	box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2),
	            0 8px 0 -3px #f6f6f6,
	            0 9px 1px -3px rgba(0, 0, 0, 0.2),
	            0 16px 0 -6px #f6f6f6,
	            0 17px 2px -6px rgba(0, 0, 0, 0.2);
}

.todo-count {
	float: left;
	text-align: left;
}

.todo-count strong {
	font-weight: 300;
}

.filters {
	margin: 0;
	padding: 0;
	list-style: none;
	position: absolute;
	right: 0;
	left: 0;
}

.filters li {
	display: inline;
}

.filters li a {
	color: inherit;
	margin: 3px;
	padding: 3px 7px;
	text-decoration: none;
	border: 1px solid transparent;
	border-radius: 3px;
}

.filters li a:hover {
	border-color: #DB7676;
}

.filters li a.selected {
	border-color: #CE4646;
}

.footer>button:last-of-type{
	display: none;
}

.clear-completed,
html .clear-completed:active {
	float: right;
	position: relative;
	line-height: 19px;
	text-decoration: none;
	cursor: pointer;
	display: block !important;;
}

.clear-completed:hover {
	text-decoration: underline;
}

.info {
	margin: 65px auto 0;
	color: #4d4d4d;
	font-size: 11px;
	text-shadow: 0 1px 0 rgba(255, 255, 255, 0.5);
	text-align: center;
}

.info p {
	line-height: 1;
}

.info a {
	color: inherit;
	text-decoration: none;
	font-weight: 400;
}

.info a:hover {
	text-decoration: underline;
}

/*
	Hack to remove background from Mobile Safari.
	Can't use it globally since it destroys checkboxes in Firefox
*/
@media screen and (-webkit-min-device-pixel-ratio:0) {
	.toggle-all,
	.todo-list li .toggle {
		background: none;
	}

	.todo-list li .toggle {
		height: 40px;
	}
}

@media (max-width: 430px) {
	.footer {
		height: 50px;
	}

	.filters {
		bottom: 10px;
	}
}

:focus,
.toggle:focus + label,
.toggle-all:focus + label {
	box-shadow: 0 0 2px 2px #CF7D7D;
	outline: 0;
}
</style>
