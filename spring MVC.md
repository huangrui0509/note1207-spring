# spring MVC

标签（空格分隔）： MVC

---
##传输数据到前台
1.使用Model进行数据传输到页面
Controller.java
```
@RequestMapping("/index")
	public String getIndex(Model model) {
		model.addAttribute("name", "小明");
		return "index";
	}
```

```
<h2 th:text="${name}"></h2>
```
2.原生request实现参数从后台传到前台页面
Controller.java
```
@RequestMapping("/testrequset")
	public String getValueByReq(HttpServletRequest request) {
		request.setAttribute("department", "后勤部");
		return "value";
	}
```

```
<h2 th:text="${department}"></h2>
```
##获取前台数据
1.直接获取参数
```
@RequestMapping(value = "/value")
public void getValue(String name,int age) {
	System.out.println("name="+name+"  age="+age);
}
```
2.使用@RequestParam
```
@RequestMapping(value = "/value2")
public void getValue2(@RequestParam(name = "name", required = true, defaultValue = "默认名字")String name,int age) {
		System.out.println("name="+name+"  age="+age);
}
```
3.原生HttpServletRequest获取页面的传值
```
@RequestMapping(value = "/value3")
public void getValue3(HttpServletRequest request) {
		String name =request.getParameter("name");
		String age =request.getParameter("age");
		System.out.println("name="+name+"  age="+age);
}
```
4.获取map对象
```
@RequestMapping(value="/testmap",method=RequestMethod.POST)
public String testMap(@RequestParam Map<String,String> map) {
		Set<Entry<String, String>> entrySet = map.entrySet();
		Iterator<Entry<String,String>> iterator = entrySet.iterator();
		while(iterator.hasNext()) {
			System.out.println(iterator.next());
		}
		return "value";	
	}
```
5.传递实体类信息
```
@RequestMapping(value="/testuser",method=RequestMethod.POST)
public String testEntity(@ModelAttribute("user") User user) {
		System.out.println(user);
		return "value";
		
}
```
6.@RequestHeader获取头信息
```
@RequestMapping(value="/testheader",method=RequestMethod.POST)
public String testHeader(@RequestParam(name="id",required=true,defaultValue="666") String id,
			@RequestHeader(name="User-Agent") String userAgent) {
		System.out.println("id="+id);
		System.out.println("userAgent="+userAgent);
		return "value";
	}
```
7.使用@PathVariable
```
//使用@PathVariable("")
//localhost:8080/testpath/11/roles/22
//直接从路径里面获取值
@RequestMapping(value="/testpath/{userId}/roles/{roleId}",method=RequestMethod.POST)
public String testPath(@PathVariable("userId") String userId,@PathVariable("roleId") String roleId ) {
		System.out.println("userId="+userId);
		System.out.println("roleId="+roleId);
		return "value";
	}
```
##将数据转成json格式
1.对象
```
@RequestMapping(value="/testjson")
@ResponseBody
public User testJson(User user) {
		user.setAge("222");
		user.setName("jjj");
		user.setHeight("122");
		return user;
		//{"name":"jjj","age":"222","height":"122"}	
	}
```
2.Map
```
@RequestMapping(value="/testjson2")
@ResponseBody
public Map<String,String> testJsonByMap(){
		Map<String,String> map = new HashMap();
		map.put("name", "sss");
		map.put("age", "123");
		return map;
		//{"name":"sss","age":"123"}
}
```