title: JSON使用技巧
date: 2018/8/5
categories:
- javascript
----------

1. JSON序列化
    ```javascript
    var xiaoming = {
        name: '小明',
        age: 14,
        gender: true,
        height: 1.65,
        grade: null,
        'middle-school': '\"W3C\" Middle School',
        skills: ['JavaScript', 'Java', 'Python', 'Lisp']
    };
    JSON.stringify(xiaoming);
    //'{"name":"明","age":14,"gender":true,"height":1.65,"grade":null,"middle-school":"\"W3C\" Middle School","skills":["JavaScript","Java","Python","Lisp"]}'
    ```
    <!--more-->
    
2. 对序列化后的字符串进行优化，缩进处理
    ```javascript
    JSON.stringify(xiaoming, null, ' ');
    ```
    输出结果
    ```json
    "{
        "name": "小明",
        "age": 14,
        "gender": true,
        "height": 1.65,
        "grade": null,
        "middle-school": "\"W3C\" Middle School",
        "skills": [ "JavaScript", "Java", "Python", "Lisp" ]
    }"
    ```
    第二个参数用于控制筛选对象的键值
3. 输出指定键值，可输入数组
    ```json
    JSON.stringify(xiaoming, ['name', 'age'], ' ');
    ```
    输出结果
    ```json
    "{
        "name": "小明",
        "age": 14
    }"
    ```
4. 还可以输入函数，进行处理
    ```json
    function convert(key, value) {
        if (typeof value === 'string') {
        	return value.toUpperCase();
        }
        return value;
    }
    JSON.stringify(xiaoming, convert, ' ');
    ```
    输出结果
    ```json
    "{
        "name": "小明",
        "age": 14,
        "gender": true,
        "height": 1.65,
        "grade": null,
        "middle-school": "\"W3C\" MIDDLE SCHOOL",
        "skills": [
            "JAVASCRIPT",
            "JAVA",
            "PYTHON",
            "LISP"
    	]
    }"
    ```
5. 还可以给`xiaoming`添加一个`toJSON`，直接序列化数据
    ```json
    var xiaoming = {
        name: '小明',
        age: 14,
        gender: true,
        height: 1.65,
        grade: null,
        'middle-school': '\"W3C\" Middle School',
        skills: ['JavaScript', 'Java', 'Python', 'Lisp'],
        toJSON: function () {
            return {
                'NAME': this.name,
                'AGE': this.age * 2
            };
    	}
    };
    JSON.stringify(xiaoming); // '{"NAME":"小明","AGE":28}'
    ```
6. JSON反序列化，使用`JSON.parse()`把内容变成一个JavaScript对象
    ```json
    JSON.parse('[1,2,3,true]'); // [1, 2, 3, true]
    JSON.parse('{"name":"小明","age":14}'); // Object {name: '小明', age: 14}
    JSON.parse('true'); // true
    JSON.parse('123.45'); // 123.45
    JSON.parse('"aa"');//"aa"
    JSON.parse('aa');//SyntaxError
    ```
7. 还可以接受函数，对键值进行处理
    ```json
    JSON.parse('{"name":"小明","age":14}', function (key, value) {
        if (key === 'name') {
        	return value + '同学';
        }
        return value;
    }); // Object {name: '小明同学', age: 14}
    ```