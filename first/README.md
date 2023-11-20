#### 学习历程
1. 绘制单点
   1. 了解使用webgl必须的元素
      1. 创建webgl上下文，设置可视区大小，通过glMatrix设置正交摄像机
      2. 初始化着色器（片元着色器和顶点着色器），创建program
      3. 初始化数据缓冲区，设置相关点坐标，区分单点设置和多点设置
         1. 单点设置webgl.vertexAttrib4fv
         2. 多点设置
            1. webgl.createBuffer();
            2. webgl.bindBuffer();
            3. webgl.bufferData();
      4. 绘制相关内容
         1. webgl.clearColor();
         2. webgl.clear();
         3. 进行绘制（可使用drawArrays和drawElements绘制）
            1. drawElements绘制比drawArrays绘制性能消耗少
   
2. 绘制动态点、绘制线、三角形

#### 了解webgl常用API - 1
   1. 类型数据
      1. 在前端页面设计时，通常使用无类型数组结构函数Array，为了处理webgl中大量的顶点数据在es2015中引入了类型数组。
      
      2. 类型化数组的出现最大的作用就是提升了数组的性能，浏览器事先知道数组中的数据类型，故而处理起来更有效率。js中array的内部实现时链表，可以动态增大减少元素，但是元素多的话，性能会比较差，类型化数组管理的是连续内存区域，知道了这块内存的起始位置，可以通过位置+N*偏移量（一次加法一次乘法操作）访问到第n个位置的元素。而array的话就需要通过链表一个一个的找下去
      
      3. 类型话数组将实现拆分为缓冲和视图两部分。一个缓冲（ArrayBuffer）描述的是内存中的一段二进制数据，缓冲没有格式可言，并且不提供机制访问其内容。为了访问在缓存对象中包含的内存，需要使用视图。视图可以将二进制数据转换为实际有类型的数组。
      
      4. 一个缓冲可以提供给多个视图进行读取，不同类型的视图读取的内存长度不同，读取出来的数据格式也不同。
      
      5. ![类型化数组](/Users/wangyue/Desktop/echo/webgl/资料/类型化数组.png)
      
      6. 类型化数组的方法、属性和常量
         1. get(index) 获取第index个元素值
         2. set(index, value) 设置第index个元素的值为value
         3. set(array, offset) 从第offset个元素开始将数组array中的值填充进数据的长度
         4. length 数组的长度
         5. BYTES_PER_ELEMENT 数组中每个元素所占的字节数
         6. 类型化数组不支持 push 和 pop方法
         7. 类型话数组创建的唯一方法就是使用new运算符 let vertics = new Float32Array([0.0, 0.5, -0.5, -0.5])
         
      7. 数据的用途不同，要求的精度和形式自然不同，比如顶点索引使用整数菊科，根据顶点的数量可以选择unit8、unit16、unit32中的哪一种整型数据，顶点的位置一般使用浮点数来表示，浮点数可以选择不同的精度表示。
      
         ![顶点数据](/Users/wangyue/Desktop/echo/webgl/资料/顶点数据.png)
      
   2. getAttribLocation(program【程序对象】, attributeName【顶点着色器程序中的顶点变量名】); 获取着色器中顶点变量的索引地址并转化为js中的变量，在gl.vertexAttribPointer方法中使用
   
   3. 顶点数据配置
      ```
        // 创建缓冲区对象
        let buffer = gl.createBuffer();
        // 绑定缓冲区对象 ARRAY_BUFFER表示顶点缓冲区；ELEMENT_ARRAY_BUFFER表示顶点索引缓冲区
        gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
        // 顶点数组data数据传入缓冲区 data表示要传入缓冲区中的数组数据 第三个参数代表通过不同的值控制传入缓冲区数据的方式、GPU使用缓冲区调用数据方式；gl.STATIC_DRAW表示静态绘制模式；gl.STREAM_DRAW表示流绘制模式；gl.DYNAMIC_DRAW表示动态绘制模式
        gl.bufferData(gl.ARRAY_BUFFER, data, gl.STATIC_DRAW);
        // 缓冲区中的数据按照一定的规律传递给位置变量apos，第一个参数为getAttribLocation返回的变量；第二个参数表示每次取用几个数据，如果设置的是1，则webgl会自动补全第2到4个变量，为0, 0, 1； 第三个参数为顶点数据类型；第四个参数表示是否归一化到区间[0, 1]或[-1, 1]; 第四个参数表示相邻两个顶点间隔的数据字节数，顶点数量再乘以该类型数据的一个元素占用的字节数； 第五个参数表示GPU从一组数据中第几个元素开始读区数据（偏移量）
        gl.vertexAttribPointer(aposLocation, 3, gl.FLOAT, false, 0, 0);
        // 允许数据传递，不设置不能正常绘制
        // 顶点缓冲区和GPU渲染管线之间存在一个硬件单元可以决定GPU是否能读取顶点缓冲区中的顶点数据，开启方法是调用以下方法；关闭的方法是disableVertexAttribArray(),两个方法的参数都是顶点着色器程序中顶点变量的索引位置
        gl.enableVertexAttribArray(aposLocation);
      ```
    4. 顶点索引配置
      ```
        // 创建缓冲区对象
        let indexBuffer = gl.createBuffer();
        // 绑定缓冲区对象
        gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indexBuffer);
        // 索引数组index数据传入缓冲区
        gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, gl.STATIC_DRAW);
      ```
    5. createBuffer()方法会在GPU控制的显存上创建一个缓冲区用来存储顶点或顶点索引数据，通过deleteBuffer(buffer)删除某个缓冲区，buffer为执行createBuffer()返回的变量

#### 了解webgl常用API - shader
1. createShader(shaderType)
   1. 创建着色器对象，接收一个着色器类型参数，标记一个着色器程序会被GPU渲染管线上哪一个着色器执行，参数shaderType是gl.VERTEX_SHADER表示该着色器程序编译后被顶点着色器执行，参数shaderType是gl.FRAGMENT_SHADER表示该着色器程序被片元着色器执行
    ```
        // 创建顶点着色器
        let vertexShader = gl.createShader(gl.VERTEX_SHADER);
        // 创建片元着色器
        let fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
    ```
2. shaderSource(shaderObject, shaderSource) 把字符串形式的顶点着色器代码、片元着色器的代码分配给各自的着色器对象
    1. shaderObject 着色器对象变量名
    2. shaderSource 字符串格式着色器程序
3. compileShader(shaderSource) 编译顶点着色器和片元着色器程序，通过参数shaderSource指定着色器程序源码
4. getShaderParameter(shaderObject, gl.COMPILE_STATUS) 获取shader中相关数据的状态值
5. getShaderInfoLog(shaderObject) 获取着色器中相关错误信息 常配合4一起调试shader
6. createPropgram() 创建程序对象，程序对象存在的意义是为了实现CUPU和GPU的通信，控制GPU着色器的工作状态，切换不同的着色器程序
7. attachShader(program, shaderObject) 绑定着色器对象到一个程序对象砂锅，每个程序对象就关联了一组顶点着色器程序、片元着色器程序，第一个参数program表示目标程序对象，第二个参数shaderObject表示要绑定的着色器对象
8. linkProgram(program)  在执行useProgram方法之前，要先连接程序对象program的顶点和片元着色器程序，检查着色程序的错误。通过连接测试后，才能通过useProgram方法把着色器程序传递给GPU，否则报错
   1. 测试项：
      1. 检查顶点、片元着色器程序中同名varying变量是否一一对应
      2. 检查顶点着色器程序中是否给verying变量赋值顶点数据
      3. 硬件资源有限，要检测attribute、uniform、varying变量的数量是否超出限制范围
9. getProgramParameter(program, value) 通过该方法可以判断linkProgram()方法是否连接成功，program参数指定要链接的程序对象，参数value定义了该方法执行后，需要反馈的数据
   1. 第二个参数对应值
      1. gl.DELETE_STATUS 是否执行deleteProgram删除程序对象
      2. gl.LINK_STATUS 程序对象是否通过linkProgram()方法链接验证 （常用）
10. useProgram(program) 调用程序对象program，执行WebGL绘制函数drawArrays()的时候，WebGL系统会把程序对象对应的顶点、片元着色器程序传递给GPU渲染管线的顶点、片元着色器功能单元。同一时刻GPU只能配置一组顶点、片元着色器程序。在代码中useProgram的特点是当再次调用方法useProgram，使用新的program程序对象作为新的参数，再次执行绘制函数的时候CPU和GPU进行通信，这时就实现了GPU着色器程序的切换，每次切换都会耗费一定的硬件资源，可以简单的类比CPU线程的切换。
   1. 一般复杂的场景都会编写多套着色器程序，放在文件中，供WebGL程序调用，比如有纹理贴图和没有纹理贴图的时候着色器程序就不同
11. delete删除对象
   1.  deleteShader(shaderObject) 如果已经执行attachshader()方法把着色器对象绑在程序对象program上，系统不会立即执行deleteShader()定义的删除操作，如果没有程序对象再使用本着色器对象，deleteShader()定义的删除操作就会执行，释放内存
   2.  deleteProgram(program) 删除程序对象，如果已经使用方法useProgram()调用了该程序对象，该方法在程序中不是立即执行，删除程序对象的原则是该程序对象不再使用，所谓不再使用就是通过方法useProgram()调用新的程序对象

#### 了解webgl常用API - 绘制
1. gl.clearColor(red, green, blue, alpha); 表示清除颜色设置为..
2. gl.clear(gl.COLOR_BUFFER_BIT) 表示把整个窗口清除为当前的清除颜色
3. gl.drawArrays(mode, first, count);
   1. mode 指定绘制图元的方式
      1. gl.POINTS 绘制一系列点
      2. gl.LINE_STRIP 绘制一条线
      3. gl.LINE_LOOP 绘制一个线圈
      4. gl.LINES 绘制一些列单独线段，每两个点作为端点，线段之间不连接
      5. gl.TRIANGLE_STRIP 绘制一个三角带
      6. gl.TRIANGLE_FAN 绘制一个三角扇
      7. gl.TRIANGLES 绘制一系列三角形，每三个点作为顶点
   2. first 指定从哪个点开始绘制
   3. count 指定绘制需要使用到多少点
4. gl.drawElements(mode, count, type, offset);
   1. mode 指定要渲染的图元类型
   2. count 指定绘制需要使用到多少点
   3. type 指定元素数组缓冲区中的值的类型，可能的值是
      1. gl.UNSIGNED_BYTE
      2. gl.UNSIGNED_SHORT
   4. offset 指定元素数组缓冲区中的偏移量，必须是给定类型大小的有效倍数