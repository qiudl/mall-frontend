前端可以使用一套系统来支持多类型用户（商户后台、供应商后台），以下是具体的实现思路和示例代码片段说明：

### 实现思路
1. 用户认证与角色管理 ：用户登录时，系统根据用户输入的信息进行认证，并确定其角色（商户或供应商）。
2. 路由控制 ：根据用户角色，动态显示不同的菜单和页面，确保不同角色只能访问其权限范围内的功能。
3. 组件复用 ：对于一些通用的组件，如表单、表格等，可以在不同角色的页面中复用，提高代码的可维护性和开发效率。
### 前端（Vue）实现示例 1. 用户认证与角色管理
在登录时，从后端获取用户角色信息，并存储在本地。

```vue
<template>
  <div>
    <input v-model="username" placeholder="用户名" />
    <input v-model="password" type="password" placeholder="密码" />
    <button @click="login">登录</button>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      username: '',
      password: ''
    };
  },
  methods: {
    login() {
      axios.post('http://localhost:8080/login', {
        username: this.username,
        password: this.password
      })
      .then(response => {
        const { role } = response.data;
        localStorage.setItem('role', role);
        this.$router.push('/dashboard');
      })
      .catch(error => {
        console.error(error);
      });
    }
  }
};
</script>
 ```
```
 2. 路由控制
根据用户角色，动态生成路由。

```javascript
import Vue from 'vue';
import VueRouter from 'vue-router';
import Dashboard from '../views/Dashboard.vue';
import SupplierProducts from '../views/SupplierProducts.vue';
import MerchantProducts from '../views/MerchantProducts.vue';

Vue.use(VueRouter);

const routes = [
  {
    path: '/',
    redirect: '/login'
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('../views/Login.vue')
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: Dashboard,
    children: [
      {
        path: 'supplier-products',
        name: 'SupplierProducts',
        component: SupplierProducts,
        meta: { requiresRole: 'supplier' }
      },
      {
        path: 'merchant-products',
        name: 'MerchantProducts',
        component: MerchantProducts,
        meta: { requiresRole: 'merchant' }
      }
    ]
  }
];

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
});

router.beforeEach((to, from, next) => {
  const role = localStorage.getItem('role');
  if (to.meta.requiresRole && to.meta.requiresRole !== role) {
    next('/login');
  } else {
    next();
  }
});

export default router;
 ```
```
 3. 组件复用
例如，商品列表组件可以在供应商和商户页面中复用。

```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <ul>
      <li v-for="product in products" :key="product.id">
        {{ product.name }} - {{ product.description }}
        <button v-if="showApproveButton" @click="approveProduct(product.id)">审核通过</button>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  props: {
    title: String,
    showApproveButton: Boolean
  },
  data() {
    return {
      products: []
    };
  },
  mounted() {
    this.getProducts();
  },
  methods: {
    getProducts() {
      const apiUrl = this.showApproveButton ? 'http://localhost:8080/products/pending' : 'http://localhost:8080/products/approved';
      axios.get(apiUrl)
        .then(response => {
          this.products = response.data;
        })
        .catch(error => {
          console.error(error);
        });
    },
    approveProduct(productId) {
      axios.put(`http://localhost:8080/products/${productId}/approve`)
        .then(response => {
          console.log(response.data);
          this.getProducts();
        })
        .catch(error => {
          console.error(error);
        });
    }
  }
};
</script>
 ```
```

### 总结
通过以上方法，前端可以使用一套系统来支持多类型用户，根据用户角色提供不同的功能和页面。同时，通过组件复用可以提高代码的可维护性和开发效率。
