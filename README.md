# grunt-kmc

[![Build Status](https://travis-ci.org/daxingplay/grunt-kmc.png?branch=master)](https://travis-ci.org/daxingplay/grunt-kmc)

[![NPM version](https://badge.fury.io/js/grunt-kmc.png)](http://badge.fury.io/js/grunt-kmc)

> [KISSY Module Compiler](https://github.com/daxingplay/kmc)的Grunt插件版本。

## 项目说明

依赖`Grunt ~0.4.1`，请首先安装Grunt，参照[Grunt安装手册](http://gruntjs.com/getting-started)和[Gruntfile样例](http://gruntjs.com/sample-gruntfile)。之后，敲入命令来安装`grunt-kmc`:

```shell
npm install grunt-kmc --save-dev
```

然后，确保你的`gruntfile.js`中载入了这个模块

```js
grunt.loadNpmTasks('grunt-kmc');
```

## 视频示例

- 生成依赖关系表：<http://asciinema.org/a/6731>
- 仅作静态合并：<http://asciinema.org/a/6732>

## Gruntfile.js 里的 KMC 任务

### 介绍

在Gruntfile.js文件中，添加名为`kmc`的任务，代码块写在`grunt.initConfig()`函数参数对象中

```js
grunt.initConfig({
	kmc: {
		options: {
			depFilePath: 'build/mods.js',
			comboOnly: true,
			fixModuleName:true,
			comboMap: true,
			packages: [
				{
					name: 'package-name',
					path: './src/',
					charset:'utf-8',
					ignorePackageNameInUri:true

				}
			],
		},
		main: {
			files: [
				{
					expand: true,
					cwd: 'src/',
					src: [ '**/*' ],
					dest: 'build/'
				}
			]
		}
	},
})
```

### 配置项

#### options.packages

- 类型: `Array`
- 默认值: `[]`

KISSY 包配置项，比如：

	packages: [
		{
			name: 'package-name',
			path: './src/',
			charset:'utf-8',
			ignorePackageNameInUri:true

		}
	],

#### options.charset

- 类型: `String`
- 默认值: `utf-8`

输入文件的编码

#### options.comboOnly

- 类型: `Boolean`
- 默认值: `false`

设置为`true`时，将不进行文件静态合并，比如两个文件`a.js`和`b.js`：

a.js

	// a.js
	KISSY.add(function(S){
		// a
	},{
		requires:['./b']
	});

b.js

	// b.js
	KISSY.add(function(S){
		// b
	});

在`comboOnly`为`false`时将静态合并，比如`a.js`将生成为：

a.js

	// b.js
	KISSY.add('pkg/b',function(S){
		// b
	});
	// a.js
	KISSY.add('pkg/a',function(S){
		// a
	},{
		requires:['./b']
	});

即所有的依赖也都合并到一个文件中。

#### options.depFilePath

- 类型: `String`
- 默认值: ``

生成依赖关系表的文件（输出）位置

#### options.depFileCharset

- 类型: `String`
- 默认值: 和`options.charset`保持一样

依赖关系表文件的编码类型

#### options.traverse

- 类型：`Boolean`
- 默认值：`false`

当指定模个文件为入口文件时，遍历子目录进行构建

#### options.fixModuleName

- 类型:`Boolean`
- 默认值:`false`

置为`true`时，会给所有文件补全模块名，建议当`comboOnly`为`true`时，总是设置此项为`true`

#### options.comboMap

- 类型：`Boolean`
- 默认值：`false`

当指定一批文件为源文件时，对这些文件只生成模块依赖关系表，存放于`options.depFilePath`中

----------------------------------

### 用法

### 示例1，单文件静态合并

入口为单个文件，将这个文件的依赖关系解析好后合并入另一个文件

	grunt.initConfig({
		kmc: {
			main:{
				options: {
					packages: [
						{
							name: 'test',
							path: 'assets/src',
							charset: 'utf-8'
						}
					]
				},
				files: [{
					// 入口和出口均为单文件
					src: 'assets/src/test/index.js',
					dest: 'assets/dist/test/index.combo.js'
				}]
			}
		}
	});

详细配置项请参照[kmc首页](https://github.com/daxingplay/kmc)。

如果输出`gbk`编码的文件，需要配置全局项

	kmc: {
		options: {
			charset:'gbk',
			packages: [
				{
					name: 'pkg-name',
					path: '../',
					charset:'gbk',
					ignorePackageNameInUri:true
				}
			]
		},
	//...
	grunt.file.defaultEncoding = 'gbk';

### 示例2，批量静态合并文件

入口为一批文件，每个文件都解析合并

	grunt.initConfig({
        kmc: {
            options: {
                packages: [
                    {
                        name: 'pkg-name',
                        path: '../',
						charset:'utf-8',
						ignorePackageNameInUri:true

                    }
                ],
				// 将 ModuleName 中的 `src` 去掉
				map: [['pkg-name/src/', 'pkg-name/']]
            },

            main: {
                files: [
                    {
						// 这里指定项目根目录下所有文件为入口文件
                        expand: true,
						cwd: 'src/',
                        src: [ '**/*.js', '!Gruntfile.js'],
                        dest: 'build/'
                    }
                ]
            }
		}
	});


### 示例3，批量静态合并，报名为变量

入口为一批文件，每个文件都解析合并，包名从配置文件中读取

	grunt.initConfig({
		// 读取`abc.json配置文件中的配置`
        pkg: grunt.file.readJSON('abc.json'),
        kmc: {
            options: {
                packages: [
                    {
                        name: '<%= pkg.name %>',
                        path: '../',
						charset:'utf-8'
                    }
                ],
				// 将 ModuleName 中的 `src` 去掉
				map: [['<%= pkg.name %>/src/', '<%= pkg.name %>/']]
            },

            main: {
                files: [
                    {
						// 这里指定项目根目录下所有文件为入口文件
                        expand: true,
						cwd: 'src/',
                        src: [ '**/*.js', '!Gruntfile.js'],
                        dest: 'build/'
                    }
                ]
            }
		}
	});

其中 abc.json 文件内容如下：

	{
		"name": "my-custom-package-name",
	}

### 示例4，针对一批文件生成依赖关系表

生成模块依赖关系表，同时源文件也被添加好模块名存放到目标目录

	grunt.initConfig({
		options: {
			packages: [
				{
					name: 'h5-test',
					path: './src/', //指定package起始路径
					charset:'utf-8',
					ignorePackageNameInUri:true
				}
			],
			// 生成模块依赖关系表
			depFilePath:'build/mods.js',
			comboOnly:true,// 不要静态合并
			fixModuleName:true,// 补全模块名称
			comboMap:true
		},
		main: {
			files: [
				{
					src: 'src/**/*.js',
					dest: 'build/'
				}
			]
		}
	});

## 更多应用案例

[Clam](http://github.com/jayli/generator-clam)工具和[ABC](http://abc.f2e.taobao.net/)依赖kmc。

## Changelog

* 0.1.7 bugfix for comboMap
* 0.1.6 add traverse option.
* 0.1.5 fix charset output bug.
