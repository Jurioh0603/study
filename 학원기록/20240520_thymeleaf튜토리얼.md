# thymeleaf튜토리얼
<https://www.thymeleaf.org/doc/tutorials/3.1/usingthymeleaf.html#strings>

## 문자, Date, LocalDateTime
 - controller
 ```
	@GetMapping("/view/ex05")
		public String ex05(Model model) {
			model.addAttribute("name","HOng이름");
			model.addAttribute("sampleVO",new SampleVO("김이름","학생",50));
			model.addAttribute("dateObj",new Date());
			model.addAttribute("localDateTImeObj",LocalDateTime.now());
			return "view/ex05";
		}
  ```

- html
 ```
	<h3>ex05.html문서</h3>
	<ul>
		<li>name = [[${name}]]<br>
		${#strings.length(name)} : [[${#strings.length(name)}]]<br>
		${#strings.isEmpty(name)} : [[${#strings.isEmpty(name)}]]<br>
		${#strings.toUpperCase(name)} : [[${#strings.toUpperCase(name)}]]<br>
		${#strings.toLowerCase(name)} : [[${#strings.toLowerCase(name)}]]<br>
		</li>
		<li>
			sampleVO = [[${sampleVO.name}]] / [[${sampleVO.nickName}]] / [[${sampleVO.age}]]<br>
			이름의 length : [[${#strings.length(sampleVO.name)}]]
		</li>
		<li>
			[[${dateObj}]]<br>
			${#dates.format(dateObj, 'dd/MMM/yyyy HH:mm')} : [[${#dates.format(dateObj, 'dd/MMM/yyyy HH:mm')}]]<br>
			${#dates.day(dateObj)} : [[${#dates.day(dateObj)}]]<br>
			${#dates.monthName(dateObj)} : [[${#dates.monthName(dateObj)}]]<br>
			${#dates.monthNameShort(dateObj)} : [[${#dates.monthNameShort(dateObj)}]]<br>
			${#dates.dayOfWeek(dateObj)} : [[${#dates.dayOfWeek(dateObj)}]]<br>
			${#dates.hour(dateObj)} : [[${#dates.hour(dateObj)}]]<br>
			${#dates.second(dateObj)} : [[${#dates.second(dateObj)}]]<br>
			${#dates.millisecond(dateObj)} : [[${#dates.millisecond(dateObj)}]]<br>
		</li>
		<li>
			[[${localDateTImeObj}]]<br>
			${#temporals.format(localDateTImeObj, 'dd/MMM/yyyy HH:mm')} : [[${#temporals.format(localDateTImeObj, 'dd/MMM/yyyy HH:mm')}]]<br>
			${#temporals.day(localDateTImeObj)} : [[${#temporals.day(localDateTImeObj)}]]<br>
		</li>
	</ul>
  ```
