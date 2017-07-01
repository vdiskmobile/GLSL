 与OpenGL ES1.x渲染管线相比，OpenGL ES 2.0渲染管线中“顶点着色器”取代了OpenGL ES 1.x渲染管线中的“变换和光照”；“片元着色器”取代了OpenGL ES 1.x渲染管线中的“纹理环境和颜色求和”、“雾”以及“Alpha测试”。 这使得开发人员在使用OpenGL ES 2.0API进行开发时，可以通过编写顶点及片元着色器程序，来完成一些顶点变换和纹理颜色计算工作，实现更加灵活、精细化的计算与渲染。

一、着色（Shader）语言

着色语言是一种类C的编程语言，但不像C语言一样支持双精度浮点型(double)、字节型(byte)、短整型(short)、长整型(long)，并且取消了C中的联合体(union)、枚举类型(enum)、无符号数(unsigned)以及位运算等特性。 着色语言中有许多内建的原生数据类型以及构建数据类型，如：浮点型(float)、布尔型(bool)、整型(int)、矩阵型(matrix)以及向量型(vec2、vec3等)等。总体来说，这些数据类型可以分为标量、向量、矩阵、采样器、结构体以及数组等。 shader支持下面数据类型：

float bool int 基本数据类型
vec2 包含了2个浮点数的向量
vec3 包含了3个浮点数的向量
vec4 包含了4个浮点数的向量
ivec2 包含了2个整数的向量
ivec3 包含了3个整数的向量
ivec4 包含了4个整数的向量
bvec2 包含了2个布尔数的向量
bvec3 包含了3个布尔数的向量
bvec4 包含了4个布尔数的向量
mat2 2*2维矩阵
mat3 3*3维矩阵
mat4 4*4维矩阵
sampler1D 1D纹理采样器
sampler2D 2D纹理采样器
sampler3D 3D纹理采样器

1. 顶点着色器

1.1 顶点着色器示例代码

uniform mat4 uMVPMatrix; // 应用程序传入顶点着色器的总变换矩阵
attribute vec4 aPosition; // 应用程序传入顶点着色器的顶点位置
attribute vec2 aTextureCoord; // 应用程序传入顶点着色器的顶点纹理坐标
attribute vec4 aColor // 应用程序传入顶点着色器的顶点颜色变量
varying vec4 vColor // 用于传递给片元着色器的顶点颜色数据
varying vec2 vTextureCoord; // 用于传递给片元着色器的顶点纹理数据
void main()
{
gl_Position = uMVPMatrix * aPosition; // 根据总变换矩阵计算此次绘制此顶点位置
vColor = aColor; // 将顶点颜色数据传入片元着色器
vTextureCoord = aTextureCoord; // 将接收的纹理坐标传递给片元着色器
}

1.2 顶点着色器介绍

顶点着色器是一个可编程的处理单元，执行顶点变换、纹理坐标变换、光照、材质等顶点的相关操作，每顶点执行一次。替代了传统渲染管线中顶点变换、光照以及纹理坐标的处理，开发人员可以根据自己的需求自行开发，大大增加了程序的灵活性。

顶点着色器主要是传入相应的Attribute变量、Uniforms变量、采样器以及临时变量，经过顶点着色器后生成Varying变量。如下图所示：

顶点着色器

（1）attribute变量(属性变量)只能用于顶点着色器中，不能用于片元着色器。 一般用该变量来表示一些顶点数据，如：顶点坐标、纹理坐标、颜色等。

（2）uniforms变量(一致变量)用来将数据值从应用程其序传递到顶点着色器或者片元着色器。 该变量有点类似C语言中的常量（const），即该变量的值不能被shader程序修改。一般用该变量表示变换矩阵、光照参数、纹理采样器等。

（3）varying变量(易变变量)是从顶点着色器传递到片元着色器的数据变量。顶点着色器可以使用易变变量来传递需要插值的颜色、法向量、纹理坐标等任意值。 在顶点与片元shader程序间传递数据是很容易的，一般在顶点shader中修改varying变量值，然后片元shader中使用该值，当然，该变量在顶点及片元这两段shader程序中声明必须是一致的 。例如：上面代码中应用程序中由顶点着色器传入片元着色器中的vColor变量。

（4）gl_Position 为内建变量，表示变换后点的空间位置。 顶点着色器从应用程序中获得原始的顶点位置数据，这些原始的顶点数据在顶点着色器中经过平移、旋转、缩放等数学变换后，生成新的顶点位置。新的顶点位置通过在顶点着色器中写入gl_Position传递到渲染管线的后继阶段继续处理。

2. 片元着色器

2.1 片元着色器示例代码

precision mediump float; // 设置工作精度
varying vec4 vColor; // 接收从顶点着色器过来的顶点颜色数据
varying vec2 vTextureCoord; // 接收从顶点着色器过来的纹理坐标
uniform sampler2D sTexture; // 纹理采样器，代表一幅纹理
void main()
{
gl_FragColor = texture2D(sTexture, vTextureCoord) * vColor;// 进行纹理采样
}
此片元着色器的主要功能为根据接收的记录片元纹理坐标的易变变量中的纹理坐标，调用texture2D内建函数从采样器中进行纹理采样，得到此片元的颜色值。最后，将采样到的颜色值传给gl_FragColor内建变量，完成片元的着色。

2.2 片元着色器介绍

片元着色器是一个处理片元值及其相关联数据的可编程单元，片元着色器可执行纹理的访问、颜色的汇总、雾化等操作，每片元执行一次。片元着色器替代了纹理、颜色求和、雾以及Alpha测试，这一部分是需要开发者自己开发的。

片元着色器

（1）varying指的是从顶点着色器传递到片元着色器的数据变量

（2）gl_FragColor为内置变量，用来保存片元着色器计算完成的片元颜色值，此颜色值将送入渲染管线的后继阶段进行处理。

二、加载着色器代码示例

private int loadShader( int shaderType, String source)
{
// 创建一个新shader
int shader = GLES20.glCreateShader(shaderType);
// 若创建成功则加载shader
if (shader != 0)
{
// 加载shader源代码
GLES20.glShaderSource(shader, source);
// 编译shader
GLES20.glCompileShader(shader);
// 存放编译成功shader数量的数组
int[] compiled = new int[1];
// 获取Shader的编译情况
GLES20.glGetShaderiv(shader, GLES20.GL_COMPILE_STATUS, compiled, 0);
if (compiled[0] == 0)
{
//若编译失败则显示错误日志并删除此shader
Log.e(TAG, "Could not compile shader " + shaderType + ":");
Log.e(TAG, GLES20.glGetShaderInfoLog(shader));
GLES20.glDeleteShader(shader);
shader = 0;
}
}

return shader;
}
上述示例代码中主要用到了三个方法：

GLES20.glCreateShader(shaderType);
GLES20.glShaderSource(shader, source);
GLES20.glCompileShader(shader);

1. GLES20.glCreateShader()， 创建一个容纳shader的容器，称为shader容器。

函数原型：
int glCreateShader (int type)
方法参数：
GLES20.GL_VERTEX_SHADER (顶点shader)
GLES20.GL_FRAGMENT_SHADER (片元shader)
如果调用成功的话，函数将返回一个整形的正整数作为shader容器的id。 如果对c++熟悉的话，该函数的返回值理解为指针或者句柄更合适。

2. GLES20.glShaderSource(shader, source)，添加shader的源代码。源代码应该以字符串数组的形式表示。当然，也可以只用一个字符串来包含所有的源代码。

函数原型：
void glShaderSource (int shader, String string)
参数含义：
shader是代表shader容器的id（由glCreateShader返回的整形数）；
string是包含源程序的字符串数组。

3. GLES20.glCompileShader(shader)，对shader容器中的源代码进行编译。

函数原型：
void glCompileShader (int shader)
参数含义：
shader是代表shader容器的id。

4. 调试

调试一个shader是非常困难的。shader的世界里没有printf，无法在控制台中打印调试信息， 更没有断点，甚至很多编辑器对shader程序关键字、变量等连高亮显示都不支持。 但是可以通过一些OpenGL提供的函数来获取编译和连接过程中的信息。

在编译阶段使用glGetShaderiv获取编译情况，在连接阶段使用glGetProgramiv获取连接情况。当错误产生的时候，还可以从InfoLog中获得更多的信息。InfoLog中存储了关于上一个操作执行时的相关信息，比如编译阶段的警告和错误，以及连接阶段产生的问题。不幸的是对于错误信息没有统一的标准，所以不同的硬件或驱动程序将提供不同的错误信息。

4.1 编译阶段使用glGetShaderiv获取编译情况

函数原型：
void glGetShaderiv (int shader, int pname, int[] params, int offset)
参数含义：
shader是一个shader的id；
pname使用GL_COMPILE_STATUS；
params是返回值，如果一切正常返回GL_TRUE代，否则返回GL_FALSE。

4.2编译阶段使用glGetShaderInfoLog获取编译错误

函数原型：
String glGetShaderInfoLog (int shader)
参数含义：
shader是一个顶点shader或者片元shader的id。

4.3 在连接阶段使用glGetProgramiv获取连接情况

函数原型：
void glGetProgramiv (int program, int pname, int[] params, int offset)
参数含义：
program是一个着色器程序的id；
pname是GL_LINK_STATUS；
param是返回值，如果一切正常返回GL_TRUE代，否则返回GL_FALSE。

4.4 在连接阶段使用glGetProgramInfoLog获取连接错误

函数原型：
String glGetProgramInfoLog (int program)
参数含义：
program是一个着色器程序的id。

4.5 清理shader的glDeleteShader方法

当不再需要某个shader或某个程序的时候，需要对其进行清理，以释放资源。前面，提到过如何向一个程序中添加一个shader。这里可调用下面的函数来将一个shader从一个程序中清除掉。

函数原型：
void glDeleteShader (int shader)；
参数含义：
shader是要被排除的顶点shader或者片元shader的id。
如果，一个shader被删除之前没有从相应的程序中排除，那么这个shader不会被实际删除，而只是被标记为被删除；当shader被从程序中排除的时候，才会被真正地删除。

三、创建着色器程序代码示例

private int createProgram(String vertexSource, String fragmentSource)
{
// 加载顶点着色器
int vertexShader = loadShader(GLES20.GL_VERTEX_SHADER, vertexSource);
if (vertexShader == 0)
{
return 0;
}

// 加载片元着色器
int pixelShader = loadShader(GLES20.GL_FRAGMENT_SHADER, fragmentSource);
if (pixelShader == 0)
{
return 0;
}

// 创建着色器程序
int program = GLES20.glCreateProgram();
// 若程序创建成功则向程序中加入顶点着色器与片元着色器
if (program != 0)
{
// 向程序中加入顶点着色器
GLES20.glAttachShader(program, vertexShader);
// 向程序中加入片元着色器
GLES20.glAttachShader(program, pixelShader);
// 链接程序
GLES20.glLinkProgram(program);
// 存放链接成功program数量的数组
int[] linkStatus = new int[1];
// 获取program的链接情况
GLES20.glGetProgramiv(program, GLES20.GL_LINK_STATUS, linkStatus, 0);
// 若链接失败则报错并删除程序
if (linkStatus[0] != GLES20.GL_TRUE)
{
Log.e(TAG, "Could not link program: ");
Log.e(TAG, GLES20.glGetProgramInfoLog(program));
GLES20.glDeleteProgram(program);
program = 0;
}
}

// 释放shader资源
GLES20.glDeleteShader(vertexShader );
GLES20.glDeleteShader(pixelShader);

return program;
}

1. glCreateProgram，在连接shader之前，首先要创建一个容纳程序的容器，称为着色器程序容器。可以通过glCreateProgram函数来创建一个程序容器。

函数原型：
int glCreateProgram ()
如果函数调用成功将返回一个正整数作为该着色器程序的id。

2. glAttachShader，接下来，我们要将shader容器添加到程序中。这时的shader容器不一定需要被编译，他们甚至不需要包含任何的代码。我们要做的只是将shader容器添加到程序中。使用glAttachShader函数来为程序添加shader容器。

函数原型：
void glAttachShader (int program, int shader)
参数含义：
program是着色器程序容器的id；
shader是要添加的顶点或者片元shader容器的id。
如果你同时拥有了，顶点shader和片元shader，需要分别将他们各自的两个shader容器添加的程序容器中。

3. glLinkProgram，链接程序。

函数原型：
void glLinkProgram (int program)
参数含义：
program是着色器程序容器的id。
在链接操作执行以后，可以任意修改shader的源代码，对shader重新编译不会影响整个程序，除非重新链接程序。

4. glUseProgram，加载并使用链接好的程序。

函数原型：
void glUseProgram (int program)
参数含义：
program是要使用的着色器程序的id。
如果将program设置为0，表示使用固定功能管线。如果程序已经在使用的时候，对程序进行重新编译，编译后的应用程序会自动替代以前的那个被调用，这时你不需要再次调用这个函数。

四、 向着色器程序中传递数据

1. 获取着色器程序内成员变量的id，也可以理解为句柄、指针。

glGetAttribLocation方法：获取着色器程序中，指定为attribute类型变量的id。

glGetUniformLocation方法：获取着色器程序中，指定为uniform类型变量的id。

如：

// 获取指向着色器中aPosition的index

maPositionHandle = GLES20.glGetAttribLocation(mProgram, "aPosition");

// 获取指向着色器中uMVPMatrix的index

muMVPMatrixHandle = GLES20.glGetUniformLocation(mProgram, "uMVPMatrix");

2. 传递数据

使用上一节获取的指向着色器相应数据成员的各个id，就能将我们自己定义的顶点数据、颜色数据等等各种数据传递到着色器当中了。

// 使用shader程序
GLES20.glUseProgram(mProgram);
// 将最终变换矩阵传入shader程序
GLES20.glUniformMatrix4fv(muMVPMatrixHandle, 1, false, MatrixState.getFinalMatrix(), 0);
// 设置缓冲区起始位置
mRectBuffer.position(0);
// 顶点位置数据传入着色器
GLES20.glVertexAttribPointer(maPositionHandle, 3, GLES20.GL_FLOAT, false, 20, mRectBuffer);
// 顶点颜色数据传入着色器中
GLES20.glVertexAttribPointer(maColorHandle, 4, GLES20.GL_FLOAT, false, 4*4, mColorBuffer);
// 顶点坐标传递到顶点着色器
GLES20.glVertexAttribPointer(maTextureHandle, 2, GLES20.GL_FLOAT, false, 20, mRectBuffer);
// 允许使用顶点坐标数组
GLES20.glEnableVertexAttribArray(maPositionHandle);
// 允许使用顶点颜色数组
GLES20.glDisableVertexAttribArray(maColorHandle);
// 允许使用定点纹理数组
GLES20.glEnableVertexAttribArray(maTextureHandle);
// 绑定纹理
GLES20.glActiveTexture(GLES20.GL_TEXTURE0);
GLES20.glBindTexture(GLES20.GL_TEXTURE_2D, texture);
// 图形绘制
GLES20.glDrawArrays(GLES20.GL_TRIANGLE_FAN, 0, 4);

2.1 glVertexAttribPointer，定义顶点属性数组。

函数原型：
void glVertexAttribPointer (int index, int size, int type, boolean normalized, int stride, Buffer ptr )
参数含义：
index 指定要修改的顶点着色器中顶点变量id；
size 指定每个顶点属性的组件数量。必须为1、2、3或者4。如position是由3个（x,y,z）组成，而颜色是4个（r,g,b,a））；
type 指定数组中每个组件的数据类型。可用的符号常量有GL_BYTE, GL_UNSIGNED_BYTE, GL_SHORT,GL_UNSIGNED_SHORT, GL_FIXED, 和 GL_FLOAT，初始值为GL_FLOAT；
normalized 指定当被访问时，固定点数据值是否应该被归一化（GL_TRUE）或者直接转换为固定点值（GL_FALSE）；
stride 指定连续顶点属性之间的偏移量。如果为0，那么顶点属性会被理解为：它们是紧密排列在一起的。初始值为0。如果normalized被设置为GL_TRUE，意味着整数型的值会被映射至区间[-1,1](有符号整数)，或者区间[0,1]（无符号整数），反之，这些值会被直接转换为浮点值而不进行归一化处理；
ptr 顶点的缓冲数据。

2.2 启用或者禁用顶点属性数组。 调用glEnableVertexAttribArray和glDisableVertexAttribArray传入参数index。如果启用，那么当glDrawArrays或者glDrawElements被调用时，顶点属性数组会被使用。

2.3 glActiveTexture， 选择活动纹理单元。

函数原型：
void glActiveTexture (int texture)
参数含义：
texture指定哪一个纹理单元被置为活动状态。texture必须是GL_TEXTUREi之一，其中0 <= i < GL_MAX_COMBINED_TEXTURE_IMAGE_UNITS，初始值为GL_TEXTURE0。
glActiveTexture()确定了后续的纹理状态改变影响哪个纹理，纹理单元的数量是依据该纹理单元所被支持的具体实现。