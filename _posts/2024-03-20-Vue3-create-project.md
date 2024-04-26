# create project
    mkdir ZsjTest.Web
    npm create vite@latest zsjtest.web -- --template vue-ts
    cd ZsjTest.Web
    npm install
    npm run dev

# add eslint
    npm intsall eslint -D
    npm init @eslint/config (注意各个选项)
    npm install -D eslint-config-airbnb-base eslint-plugin-import
## add lint script
    在package.json文件中的script中添加lint命令
    {
        "scripts": {
            // eslint . 为指定lint当前项目中的文件
            // --ext 为指定lint哪些后缀的文件
            // --fix 开启自动修复
            "lint": "eslint . --ext .vue,.js,.ts,.jsx,.tsx --fix"
        }
    }
## new-item .eslintignore
    内容如下：
    build/
    config/
    contents/
    node_modules/
    dist/
    .eslintrc.cjs
    public/
    vite.config.ts
    src/components.d.ts
    src/auto-imports.d.ts
    .eslintrc-auto-import.json

## change .eslintrc.cjs
    增加setting项，import时可以不输入文件扩展名
    "settings": {
      "import/resolver": {
        "node": {
          "extensions": [".js", ".jsx", ".ts", ".tsx", ".d.ts"]
        }
      }
    },
    在extend部分添加一项 "airbnb-base"
    rules部分设置为以下内容：
    'rules': {
      '@typescript-eslint/ban-types': [
        'error',
        {
          'extendDefaults': true,
          'types': {
            '{}': false
          }
        }
      ],
      '@typescript-eslint/no-explicit-any': ['off'],
      "import/extensions": [
        "error",
        "ignorePackages",
        {
          "js": "never",
          "jsx": "never",
          "ts": "never",
          "tsx": "never"
        }
    ],
    "linebreak-style": 0, //不检查行尾换行符
    // "no-extra-semi": 0, //不检查多余的分号
    // "@typescript-eslint/no-extra-semi": 0, //不检查typescript中多余的分号
    "no-console": 0, //不禁用console
    "no-param-reassign": [
        "error",
        {
          "props": true,
          "ignorePropertyModificationsFor": [
            "config", // for axios config
          ]
        }
      ],
      // typescript下的no-shadow和javascript不一致
      "no-shadow": "off",
      "@typescript-eslint/no-shadow": ["error"],
      // 设置emit时需要以下设置
      'no-unused-vars': 'off',
      '@typescript-eslint/no-unused-vars': ['error', { vars: 'all', args: 'after-used', ignoreRestSiblings: true }],
      "max-len": ["error", {"code": 120, "ignoreTemplateLiterals": true, "ignoreStrings": true, "ignorePattern": 'd="([\\s\\S]*?)"'}],
    }

## new-item .editorconfig
    内容如下：
    root = true

    [*]
    charset = utf-8
    indent_style = space
    indent_size = 2
    end_of_line = lf
    insert_final_newline = true
    trim_trailing_whitespace = true

## new-item .vscode\settings.json
    内容如下：
    {
        // 开启自动修复
        "editor.codeActionsOnSave": {
          "source.fixAll": "never",
          "source.fixAll.eslint": "explicit"
        }
    }

# add Tailwind CSS
    npm install -D tailwindcss postcss autoprefixer
    npx tailwindcss init -p --ts

## postcss.config.js内容如下
    export default {
      plugins: {
        tailwindcss: {},
        autoprefixer: {},
      },
    };
## tailwind.config.ts内容如下
    import type { Config } from 'tailwindcss';
    // eslint-disable-next-line import/no-extraneous-dependencies
    import { fontFamily } from 'tailwindcss/defaultTheme';
    /*
    在index.html页面head部分加入网页字体信息：
    <link href="https://fonts.googlefonts.cn/css?family=Source+Sans+Pro" rel="stylesheet">
    这里的fonts.googlefonts.cn是googlefont在国内的站点，可以进入https://googlefonts.cn/ 查询对应字体使用方法
    */

    export default {
      content: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}'],
      theme: {
        fontFamily: {
          sans: [
            '"Source Sans Pro"',
            ...fontFamily.sans,
          ],
        },
        extend: {},
      },
      plugins: [],
      darkMode: 'class',
    } satisfies Config;

## 在src下增加tailwind.css文件，内容如下，在main.ts中引入tailwind.css
    @tailwind base;
    @tailwind components;
    @tailwind utilities;

    body {
      --el-color-primary: #01916d;
      --el-color-primary-dark-2: #01815d;
      --el-color-primary-light-3: #01512d;
    }

    .el-main {
      padding: 0 !important;
    }

    /* 修复tailwindcss 与element plus按钮背景的冲突*/
    .el-button {
      background-color: var(--el-button-bg-color);
    }
    /*
    也可以在tailwind.config.ts中增加以下配置
    corePlugins: {
      preflight: false,
    },
    这个配置会不加载tailwind css中的预设置css, 即preflight.css，可以去UNPKG下载tailwind对应版本的preflight.css
    删除以下样式中的background-color: transparent; /* 2 */，然后在main.ts中直接引入这个preflight.css：
    button,
    [type='button'],
    [type='reset'],
    [type='submit'] {
      -webkit-appearance: button; /* 1 */
      background-color: transparent; /* 2 */
      background-image: none; /* 2 */
    }
    */

# 引入Element Plus, 推荐使用自动导入方式，但初期可以使用手动导入，以了解具体过程
    npm install element-plus
    npm install unplugin-element-plus -D

## 修改vite.config.ts, 增加以下内容，以便能引入样式
    import ElementPlus from 'unplugin-element-plus/vite'

    export default defineConfig({
      // ...
      plugins: [vue(), ElementPlus({})],
    })

## 如果使用 unplugin-element-plus 并且只使用组件 API，你需要手动导入样式。

    Example:

    import 'element-plus/es/components/message/style/css'
    import { ElMessage } from 'element-plus'

## Element Plus 黑暗模式
    main.ts在其他样式之前引入以下内容：
    import 'element-plus/theme-chalk/dark/css-vars.css';

## Element Plus配置语言
  App.vue中引入以下内容：
  import zhCn from 'element-plus/es/locale/lang/zh-cn';
  import { ElConfigProvider } from 'element-plus';

  <template>
    <el-config-provider :locale="zhCn">
      ......
    </el-config-provider>
  </template>

# 增加Element Plus Icon
    npm install @element-plus/icons-vue
    使用方式如下:
    <script lang="ts" setup>
    import { ElIcon } from 'element-plus';
    import { Sunny } from '@element-plus/icons-vue';
    </script>
    <template>
      <el-icon><Sunny /></el-icon>
    </template>

# 增加pinia
    npm install pinia

## main.ts增加以下内容：
    import { createPinia } from 'pinia';
    createApp(App).use(createPinia()).mount('#app');

## 在src下创建stores目录， 增加useUserInfoStore.ts文件，内容如下：
    // 对于pinia store， 如果使用解构赋值，会导致基本类型属性失去响应性，
    // 要么使用pinia提供的storeToRefs来包装store, 要么直接如下面示例一样,
    // 始终直接****Store.XXXX来操作，个人建议这样操作，这种情况下，
    // 不用写.value, 并且变量来源更明确

    // 有些需要持久化的store(即用户F5刷新页面时仍然要保持对应值)，可以使用
    // pinia-plugin-persistedstate， 注意此时store中必须使用ref, 不能使用reactive

    import { defineStore } from 'pinia';
    import { ref } from 'vue';
    import type UserInfoDto from '../types/UserInfoDto';

    const useUserInfoStore = defineStore('UserInfoStore', () => {
      console.log('now in useUserInfoStore');
      const emptyUserInfo : UserInfoDto = {
        userId: 0,
        userNumber: null,
        engLastName: null,
        engMidName: null,
        engFirstName: null,
        nativeName: null,
        email: null,
      };
      const userInfo = ref<UserInfoDto>(emptyUserInfo);
      const resetUserInfo = () => {
        userInfo.value = emptyUserInfo;
      };
      const setUserInfo = (tempUserInfo : UserInfoDto) => {
        console.log(`setuSERInfo tempUserInfo: ${JSON.stringify(tempUserInfo)}`);
        userInfo.value = tempUserInfo;
      };

      return { userInfo, resetUserInfo, setUserInfo };
    });

    export default useUserInfoStore;


# 增加vue-router
    npm install vue-router

## 在src下创建views目录, 在views目录下创建TestView.vue文件，用以测试，内容如下：
    <template>
      <h1>Test View</h1>
    </template>

## 在src下创建router目录，在router目录下增加index.ts文件，内容如下：
    import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router';

    const routes:Array<RouteRecordRaw> = [
      {
        path: '/',
        name: 'Test',
        component: () => import('../views/TestView.vue'),
        meta: {
          title: '测试',
        },
      },
    ];

    // 配置router对象
    const router = createRouter({
      // 如果不是部署在根目录，需要添加base参数
      history: createWebHistory(),
      routes,
    });
    const defaultTitle = '富士胶片商业创新（中国）有限公司';

    router.beforeEach((to, _from, next) => {
      document.title = to.meta.title ? `${defaultTitle}-${to.meta.title}` : defaultTitle;
      next();
    });

    export default router;


## main.ts增加以下内容：
    import router from './router/index.ts';
    createApp(App).use(createPinia()).use(router).mount('#app');

# 修改vite.config.js, 针对build增加以下设置

      base: './',
      esbuild: {
        drop: ['console', 'debugger']
      },
      build: {
        rollupOptions: {
          external: ['vue', 'vue-demi'],
          output: {
            chunkFileNames: 'js/[name]-[hash].js',
            entryFileNames: 'js/[name]-[hash].js',
            manualChunks(id) {
              if (id.indexOf('node_modules') !== -1) {
                let newId = id.toString();
                if (newId.indexOf('/.pnpm/') !== -1) {
                  newId = newId.replace('/.pnpm/', '/');
                }
                // 部分较大的包单独做为一个资源，其他的作为一个资源
                if (newId.toString().split('node_modules/')[1].split('/')[0].indexOf('element-plus') !== -1) {
                  return 'element-plus';
                } else if (newId.toString().split('node_modules/')[1].split('/')[0].indexOf('vue') !== -1) {
                  return 'vue';
                } else if (newId.toString().split('node_modules/')[1].split('/')[0].indexOf('xlsx') !== -1) {
                  return 'xlsx';
                }
                return 'vendor';
              } else {
                return 'index';
              }
            }
          },
        }
      }

# 针对一些包使用cdn的处理方式
Element Plus对应文件较大，需要衡量是否通过CDN引入，如果使用了大部分的Element Plus组件，可以通过CDN引入，如果只使用了少数几种Element Plus组件，直接使用按需引入来打包会更小

## 安装vite-plugin-html， rollup-plugin-external-globals
    pnpm install -D vite-plugin-html rollup-plugin-external-globals

## 在vite.config.ts中引入, 现在vite.config.ts内容如下：
    import { PluginOption, defineConfig } from 'vite'
    import vue from '@vitejs/plugin-vue'
    import ElementPlus from 'unplugin-element-plus/vite'
    import { createHtmlPlugin } from 'vite-plugin-html'
    import externalGlobals from 'rollup-plugin-external-globals';

    // https://vitejs.dev/config/
    export default (mode: any) => {
      console.log('mode: ' + JSON.stringify(mode));
      const plugins : PluginOption[] = [
        vue(),
        ElementPlus({}),
      ];

      if (mode.mode === 'production') {
        // 不要在css文件中出现注释，会影响rollup
        plugins.push(externalGlobals({
          vue: "Vue",
          "vue-demi": "VueDemi",
          "element-plus": "ElementPlus",
        }));
        // 此部分可以不用，生成后在index.html中手工添加
        plugins.push(createHtmlPlugin({
          template: './index.html',
          inject: {
            tags:[
              {
                injectTo: 'head',
                tag: 'script',
                attrs: {
                  src: 'https://cdn.bootcdn.net/ajax/libs/vue/3.3.4/vue.global.min.js',
                  defer: true
                }
              },
              {
                // 使用了pinia时，使用cdn引入vue, 就必须引入vue-demi, pinia需要依据vue-demi判断vue的版本
                injectTo: 'head',
                tag: 'script',
                attrs: {
                  src: 'https://cdn.bootcdn.net/ajax/libs/vue-demi/0.14.6/index.iife.min.js',
                  defer: true
                }
              },
              {
                injectTo: 'head',
                tag: 'script',
                attrs: {
                  src: 'https://cdn.bootcdn.net/ajax/libs/element-plus/2.3.12/index.full.min.js',
                  defer: true
                }
              },
              // 由于css覆盖的需要，elementplus对应的css文件需要放在其他css之后，因此放在body的开头
              {
                injectTo: 'body-prepend',
                tag: 'link',
                attrs: {
                  rel: 'stylesheet',
                  crossorigin: true,
                  href: 'https://cdn.bootcdn.net/ajax/libs/element-plus/2.3.12/index.css',
                }
              },
            ]
          }
        }));
      };

      return defineConfig({
        plugins: plugins,
        base: './',
        esbuild: {
          drop: ['console', 'debugger']
        },
        build: {
          rollupOptions: {
            external: ['vue', 'vue-demi', 'element-plus'],
            output: {
              chunkFileNames: 'js/[name]-[hash].js',
              entryFileNames: 'js/[name]-[hash].js',
              manualChunks(id) {
                if (id.indexOf('node_modules') !== -1) {
                  let newId = id.toString();
                  if (newId.indexOf('/.pnpm/') !== -1) {
                    newId = newId.replace('/.pnpm/', '/');
                  }
                  // 部分较大的包单独做为一个资源，其他的作为一个资源
                  if (newId.toString().split('node_modules/')[1].split('/')[0].indexOf('element-plus') !== -1) {
                    return 'element-plus';
                  } else if (newId.toString().split('node_modules/')[1].split('/')[0].indexOf('vue') !== -1) {
                    return 'vue';
                  } else if (newId.toString().split('node_modules/')[1].split('/')[0].indexOf('xlsx') !== -1) {
                    return 'xlsx';
                  }
                  return 'vendor';
                } else {
                  return 'index';
                }
              }
            },
          },
        }
      })
    }


# 使用iis express启动网站测试打包后的站点
    进入目录"C:\Program Files\IIS Express\", 使用powershell运行
    .\iisexpress.exe /path:F:\source\ZsjTestApiWithVue\ZsjTest.Web\dist\ /port:9090
