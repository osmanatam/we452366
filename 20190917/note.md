## webpack

### webpack-optimize

#### purgecss-webpack-plugin
- purgecss
- 可以去除未使用的 css，一般与 glob、glob-all 配合使用
- 必须和mini-css-extract-plugin配合使用
- paths路径是绝对路径
npm i -D purgecss-webpack-plugin glob

webpack.config.js

const glob = require('glob');
const PurgecssPlugin = require('purgecss-webpack-plugin');

module.exports = {
  mode: 'development',
  plugins: [
    new PurgecssPlugin({
      paths: glob.sync(`${path.join(__dirname, 'src')}/**/*`, 
      {nodir: true}), // 不匹配目录，只匹配文件
    }),
  ],
}

#### DLL
.dll为后缀的文件称为动态链接库，在一个动态链接库中可以包含给其他模块调用的函数和数据

把基础模块独立出来打包到单独的动态连接库里
当需要导入的模块在动态连接库里的时候，模块不能再次被打包，而是去动态连接库里获取
dll-plugin

##### 定义Dll
DllPlugin插件： 用于打包出一个个动态连接库
DllReferencePlugin: 在配置文件中引入DllPlugin插件打包好的动态连接库
const path=require('path');
const DllPlugin=require('webpack/lib/DllPlugin');
module.exports={
    entry: {
        react:['react','react-dom']
    },// 把 React 相关模块的放到一个单独的动态链接库
    output: {
        path: path.resolve(__dirname,'dist'),// 输出的文件都放到 dist 目录下
        filename: '[name].dll.js',//输出的动态链接库的文件名称，[name] 代表当前动态链接库的名称
        library: '_dll_[name]',//存放动态链接库的全局变量名称,例如对应 react 来说就是 _dll_react
    },
    plugins: [
        new DllPlugin({
            // 动态链接库的全局变量名称，需要和 output.library 中保持一致
            // 该字段的值也就是输出的 manifest.json 文件 中 name 字段的值
            // 例如 react.manifest.json 中就有 "name": "_dll_react"
            name: '_dll_[name]',
            // 描述动态链接库的 manifest.json 文件输出时的文件名称
            path: path.join(__dirname, 'dist', '[name].manifest.json')
        })
    ]
}
webpack --config webpack.dll.config.js --mode development

##### 使用动态链接库文件
const DllReferencePlugin = require('webpack/lib/DllReferencePlugin')
plugins: [
  new DllReferencePlugin({
    manifest:require('./dist/react.manifest.json')
  })
]
webpack --config webpack.config.js --mode development

##### html中使用
<script src="react.dll.js"></script>
<script src="bundle.js"></script>

#### 多进程处理

##### HappyPack
构建需要解析和处理文件,文件读写和计算密集型的操作太多后速度会很慢
Node.js 之上的 Webpack 是单线程模型
happypack 就能让Webpack把任务分解给多个子进程去并发的执行，子进程处理完后再把结果发送给主进程。
npm i happypack@next -D
const HappyPack = require('happypack');
    rules: [
    {
        test: /\.js$/,
        // 把对 .js 文件的处理转交给 id 为 babel 的 HappyPack 实例
        use: ['happypack/loader?id=babel'],
        exclude: path.resolve(__dirname, 'node_modules'),
    },
    {
        test: /\.css$/,
        // 把对 .css 文件的处理转交给 id 为 css 的 HappyPack 实例
        use: ['happypack/loader?id=css']
    }
]
new Happypack({
    //ID是标识符的意思，ID用来代理当前的happypack是用来处理一类特定的文件的
    id: 'js',
    use: [{
        loader: 'babel-loader',
        //options=query都是向插件传递参数的
        options: {
            presets: [["@babel/preset-env", { modules: false }], "@babel/preset-react"],
            plugins: [
                ["@babel/plugin-proposal-decorators", { "legacy": true }],
                ["@babel/plugin-proposal-class-properties", { "loose": true }],
            ]
        }
    }]
}),
new Happypack({
    //ID是标识符的意思，ID用来代理当前的happypack是用来处理一类特定的文件的
    id: 'css',
    use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader'],
    threads: 4,//你要开启多少个子进程去处理这一类型的文件
    verbose: true//是否要输出详细的日志 verbose
})

##### thread-loader
把这个 loader 放置在其他 loader 之前， 放置在这个 loader 之后的 loader 就会在一个单独的 worker 池(worker pool)中运行
thread-loader
{
        test: /\.(js)$/,
        use: [
           {
            loader:'thread-loader',
            options:{
              workers:3
            }
          }, 
          {
            loader:'babel-loader'
          }
        ],
      }

##### webpack-parallel-uglify-plugin
webpack默认提供了UglifyJS插件来压缩JS代码，但是它使用的是单线程压缩代码，也就是说多个js文件需要被压缩，它需要一个个文件进行压缩。所以说在正式环境打包压缩代码速度非常慢(因为压缩JS代码需要先把代码解析成用Object抽象表示的AST语法树，再去应用各种规则分析和处理AST，导致这个过程耗时非常大)。
当webpack有多个JS文件需要输出和压缩时候，原来会使用UglifyJS去一个个压缩并且输出，但是ParallelUglifyPlugin插件则会开启多个子进程，把对多个文件压缩的工作分别给多个子进程去完成，但是每个子进程还是通过UglifyJS去压缩代码。无非就是变成了并行处理该压缩了，并行处理多个子任务，效率会更加的提高

webpack-parallel-uglify-plugin
cnpm i webpack-parallel-uglify-plugin -D
let ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin');
new ParallelUglifyPlugin({});

#### CDN

CDN 又叫内容分发网络，通过把资源部署到世界各地，用户在访问时按照就近原则从离用户最近的服务器获取资源，从而加速资源的获取速度。
cdn

HTML文件不缓存，放在自己的服务器上，关闭自己服务器的缓存，静态资源的URL变成指向CDN服务器的地址
静态的JavaScript、CSS、图片等文件开启CDN和缓存，并且文件名带上HASH值
为了并行加载不阻塞，把不同的静态资源分配到不同的CDN服务器上

##### 使用缓存
由于 CDN 服务一般都会给资源开启很长时间的缓存，例如用户从 CDN 上获取到了 index.html 这个文件后， 即使之后的发布操作把 index.html 文件给重新覆盖了，但是用户在很长一段时间内还是运行的之前的版本，这会新的导致发布不能立即生效 解决办法
针对 HTML 文件：不开启缓存，把 HTML 放到自己的服务器上，而不是 CDN 服务上，同时关闭自己服务器上的缓存。自己的服务器只提供 HTML 文件和数据接口。
针对静态的 JavaScript、CSS、图片等文件：开启 CDN 和缓存，上传到 CDN 服务上去，同时给每个文件名带上由文件内容算出的 Hash 值
带上 Hash 值的原因是文件名会随着文件内容而变化，只要文件发生变化其对应的 URL 就会变化，它就会被重新下载，无论缓存时间有多长。
启用CDN之后 相对路径，都变成了绝对的指向 CDN 服务的 URL 地址

##### 域名限制
同一时刻针对同一个域名的资源并行请求是有限制
可以把这些静态资源分散到不同的 CDN 服务上去
多个域名后会增加域名解析时间
可以通过在 HTML HEAD 标签中 加入<link rel="dns-prefetch" href="http://img.zhufengpeixun.cn">去预解析域名，以降低域名解析带来的延迟

##### 接入CDN
要给网站接入 CDN，需要把网页的静态资源上传到 CDN 服务上去，在服务这些静态资源的时候需要通过 CDN 服务提供的 URL 地址去访问

output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name]_[hash:8].js',
    publicPath: 'http://img.zhufengpeixun.cn'
},

#### Tree Shaking
一个模块可以有多个方法，只要其中某个方法使用到了，则整个文件都会被打到bundle里面去，tree shaking就是只把用到的方法打入bundle,没用到的方法会uglify阶段擦除掉
原理是利用es6模块的特点,只能作为模块顶层语句出现,import的模块名只能是字符串常量

##### 开启

webpack默认支持，在.babelrc里设置module:false即可在production mode下默认开启
The 'modules' option must be one of 'false' to indicate no module processing .babelrc
还要注意把devtool设置为null
"presets":[
    ["@babel/preset-env",{"modules":true}],//转译 ES6 ES7
    "@babel/preset-react"//转译JSX语法
],

##### 没有导入和使用
functions.js

function func1(){
  return 'func1';
}
function func2(){
     return 'func2';
}
export {
  func1,
  func2
}
import {func2} from './functions';
var result2 = func2();
console.log(result2);

##### 代码不会被执行，不可到达
if(false){
 console.log('false')
}

##### 代码执行的结果不会被用到
import {func2} from './functions';
func2();

##### 代码中只会影响死变量，只写不读
var aabbcc='aabbcc';
aabbcc='eeffgg';

#### 代码分割

##### 代码分割的意义
对于大的Web应用来讲，将所有的代码都放在一个文件中显然是不够有效的，特别是当你的某些代码块是在某些特殊的时候才会被用到。
webpack有一个功能就是将你的代码库分割成chunks语块，当代码运行到需要它们的时候再进行加载。 适用的场景
抽离相同代码到一个共享块
脚本懒加载，使得初始下载的代码更小

##### Entry Points
Entry Points：入口文件设置的时候可以配置
这种方法的问题
如果入口 chunks 之间包含重复的模块(lodash)，那些重复模块都会被引入到各个 bundle 中
不够灵活，并且不能将核心应用程序逻辑进行动态拆分代码
entry: {
        index: "./src/index.js",
        login: "./src/login.js"
}

##### 动态导入和懒加载

用户当前需要用什么功能就只加载这个功能对应的代码，也就是所谓的按需加载 在给单页应用做按需加载优化时，一般采用以下原则：
对网站功能进行划分，每一类一个chunk
对于首次打开页面需要的功能直接加载，尽快展示给用户,某些依赖大量代码的功能点可以按需加载
被分割出去的代码需要一个按需加载的时机
动态import 目前并没有原生支持，需要babel
cnpm i @babel/plugin-syntax-dynamic-import --save-dev
{plugins:["@babel/plugin-syntax-dynamic-import"]}
hello.js

module.exports = "hello";
index.js

document.querySelector('#clickBtn').addEventListener('click',() => {
    import('./hello').then(result => {
        console.log(result.default);
    });
});
index.html

<button id="clickBtn">点我</button>

##### 提取公共代码

###### 为什么需要提取公共代码

大网站有多个页面，每个页面由于采用相同技术栈和样式代码，会包含很多公共代码，如果都包含进来会有问题

相同的资源被重复的加载，浪费用户的流量和服务器的成本；
每个页面需要加载的资源太大，导致网页首屏加载缓慢，影响用户体验。
如果能把公共代码抽离成单独文件进行加载能进行优化，可以减少网络传输流量，降低服务器成本

###### 如何提取

基础类库，方便长期缓存
页面之间的公用代码
各个页面单独生成文件
文档
common-chunk-and-vendor-chunk
webpack将会基于以下条件自动分割代码块:
新的代码块被共享或者来自node_modules文件夹
新的代码块大于30kb(在min+giz之前)
按需加载代码块的请求数量应该<=5
页面初始化时加载代码块的请求数量应该<=3
默认配置

  optimization: {
    // 这里放着优化的内容
    minimizer: [
      // 表示放优化的插件
      new TerserWebpackPlugin({
               parallel:true,//开启多进程并行压缩
               cache:true//开启缓存
      }),
      new OptimizeCssAssetsWebpackPlugin({
        assetNameRegExp: /\.css$/g, // 指定要压缩的模块的正则
        // cssnano是PostCSS的CSS优化和分解插件。cssnano采用格式很好的CSS，并通过许多优化，以确保最终的生产环境尽可能小。
        cssProcessor: require('cssnano'),
      }),
    ],
    splitChunks: {
        chunks: "all",//默认作用于异步chunk，值为all/initial/async
        minSize: 30000,  //默认值是30kb,代码块的最小尺寸
        minChunks: 1,  //被多少模块共享,在分割之前模块的被引用次数
        maxAsyncRequests: 5,  //按需加载最大并行请求数量
        maxInitialRequests: 3,  //一个入口的最大并行请求数量
        name: true,  //打包后的名称，默认是chunk的名字通过分隔符（默认是～）分隔开，如vendor~
        automaticNameDelimiter:'~',//默认webpack将会使用入口名和代码块的名称生成命名,比如 'vendors~main.js'
        cacheGroups: { //设置缓存组用来抽取满足不同规则的chunk,下面以生成common为例
            vendors: {
              chunks: "initial",
              name: 'vendors',  //可以通过'name'配置项来控制切割之后代码块的命名,给多个分割之后的代码块分配相同的名称,所有的vendor 模块被放进一个共享的代码块中,不过这会导致多余的代码被下载所以并不推荐
              test: /node_modules/,//条件
              priority: -10 ///优先级，一个chunk很可能满足多个缓存组，会被抽取到优先级高的缓存组中,为了能够让自定义缓存组有更高的优先级(默认0),默认缓存组的priority属性为负值.
            },
             commons: {
              chunks: "initial",
              name: 'commons',
              minSize: 0,//最小提取字节数
              minChunks: 1, //最少被几个chunk引用
              priority: -20,
              reuseExistingChunk: true//    如果该chunk中引用了已经被抽取的chunk，直接引用该chunk，不会重复打包代码
            }
      }
    },
    //runtime包含:在模块交互时,连接模块所需的加载和解析逻辑。包括浏览器中的已加载模块的连接，以及懒加载模块的执行逻辑.
    //设置optimization.runtimeChunk=true ,将每一个入口添加一个只包含runtime的额外代码块.然而设置值为single,只会为所有生成的代码块创建一个共享的runtime文件.runtime:连接模块化应用程序的所有代码.
    runtimeChunk:{
        name:'manifest'
    }
  },

###### 提取公共代码

pageA.js

import utils1 from './utils1';
import utils2 from './utils2';
import $ from 'jquery';
console.log(utils1,utils2,$);
pageB.js

import utils1 from './utils1';
import utils2 from './utils2';
import $ from 'jquery';
console.log(utils1,utils2,$);;
pageC.js

import utils3 from './utils3';
import utils1 from './utils1';
import $ from 'jquery';
console.log(utils1,utils3,$);

    entry: {
        pageA: './src/pageA',
        pageB: './src/pageB',
        pageC: './src/pageC'
    },
    output: {
        path: path.resolve(__dirname,'dist'),
        filename: '[name].js'
    },
  plugins:[
       new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'pageA.html',
            chunks: ['pageA'],
            minify: {
                removeAttributeQuotes: true
            }
        }),
        new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'pageB.html',
            chunks: ['pageB'],
            minify: {
                removeAttributeQuotes: true
            }
        }),


        new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'pageC.html',
            chunks: ['pageC'],
            minify: {
                removeAttributeQuotes: true
            }
        })
    ]
splitchunks