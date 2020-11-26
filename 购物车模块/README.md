## 购物车模块思想

首先，购物车分为两种情况，一种是登录状态下的购物车，另一种是未登录状态下的购物车。

#### 1.添加购物车

未登录的情况下我们添加购物车选择的是添加到Cookie中，持久化时间是一天，因为信息是无法存放到Redis中的，我们只能选择存放在客户端。

```
// 未登录 ,操作Cookie中的购物车数据
                // 1、获取购物车中已有的数据，
                String cartJson = CookieUtils.getCookieValue(request, USIAN_CART, true);
                // 创建一个购物车
                HashMap<String, TbItem> carMap;
                if (cartJson == null || "".equals(cartJson)) {
                    carMap = new HashMap<>();
                } else {
                    carMap = (HashMap<String, TbItem>) JsonUtils.jsonToMap(cartJson, TbItem.class);
                }
                // 2、商品添加到购物查中
                // 判断是否ItemId商品是否已经加入到购物车
                // 如果存在就修改数量
                if (carMap.containsKey(itemId)) {
                    TbItem tbItem = carMap.get(itemId);
                    tbItem.setNum(tbItem.getNum() + counter);
                } else {
                    // 如果不存在就去数据库中查找
                    TbItem tbItem = portalFeign.selectItemInfo(Long.parseLong(itemId));
                    tbItem.setNum(counter);
                    carMap.put(itemId, tbItem);
                }
                CookieUtils.setCookie(request, response, USIAN_CART, JsonUtils.objectToJson(carMap), true);
```

另一种情况就是登录状态下的购物车，这个时候我们就把购物车信息存放到Redis中。

```
//首先从Redis获取商品信息
        Map<String, TbItem> resultMap = (Map<String, TbItem>) redisClient.hget(USIAN_CART, userId);
        if (resultMap == null || resultMap.size() <= 0) {
            //不存在从数据库中查询商品信息
            resultMap = new HashMap<>();
            TbItem tbItem = tbItemMapper.selectByPrimaryKey(Long.parseLong(itemId));
            tbItem.setNum(counter);
            resultMap.put(itemId, tbItem);
        } else {
            if (resultMap.containsKey(itemId)) {
                TbItem tbItem = resultMap.get(itemId);
                System.out.println(counter);
                System.out.println(itemId);
                tbItem.setNum(tbItem.getNum() + counter);
            } else {
                TbItem tbItem = tbItemMapper.selectByPrimaryKey(Long.parseLong(itemId));
                tbItem.setNum(counter);
                resultMap.put(itemId, tbItem);
            }
        }
        redisClient.hset(USIAN_CART, userId, resultMap);
```

我们将信息存放到Redis中，键是用户的id，这样的话每次登陆就可以向Redis发出请求，根据用户id查询到购物车信息。

#### 2.展示购物车

展示购物车也分两种情况：

未登录的情况下我们去请求Cookie中的数据，然后取值转换为List集合，返回给前台。

```
if (cookieJson != null && !"".equals(cookieJson)) {
    resultMap = (HashMap<String, TbItem>) JsonUtils.jsonToMap(cookieJson, TbItem.class);
    } else {
    resultMap = new HashMap<>();
    }
    for (String s : resultMap.keySet()) {
    TbItem tbItem = resultMap.get(s);
    System.out.println(tbItem.toString());
    tbItems.add(tbItem);
    }
```

因为未登录的情况下我们存储的键为商品id，所以需要根据依次取出value放入到List中。



登录的情况下就需要从Redis中获取数据了，

```
Map<String, TbItem> resultMap = (Map<String, TbItem>) redisClient.hget(USIAN_CART, userId);
ArrayList<TbItem> list = new ArrayList<>();
if (resultMap != null && resultMap.size() > 0) {
    for (String s : resultMap.keySet()) {
        TbItem tbItem = resultMap.get(s);
        list.add(tbItem);
    }
}
```

思路就是拿到Redis中的Map集合，然后根据所有的key去获取value，最后也需要存放到List集合当中去。

#### 3.修改商品数量

在我们加入购物车商品后，增加了一下商品的数量，但是如果我们这时候去刷新页面，发现商品的数量还是回滚到了原来是值，这就是因为我们不管有没有登录，取值的地方没有发生改变，所以我们需要写一个修改商品数量并且同步到Cookie或者Redis中的方法。

如果未登录，同步到Cookie

```
String cookieJson = CookieUtils.getCookieValue(request, USIAN_CART, true);
HashMap<String, TbItem> resultMap = new HashMap<>();
resultMap = (HashMap<String, TbItem>) JsonUtils.jsonToMap(cookieJson, TbItem.class);
if (resultMap != null && resultMap.size() > 0) {
    //cookie有内容
    if (resultMap.containsKey(itemId)) {
        TbItem tbItem = resultMap.get(itemId);
        tbItem.setNum(num);
    }
    //将数据会写给Cookie，因为有中文所以需要编码
    CookieUtils.setCookie(request, response, USIAN_CART, JsonUtils.objectToJson(resultMap), true);
```

根据Cookie获取到数据，如果key与传入的商品id相同，那么就去修改商品的数量。

但是如果没有登录，我们就需要将数据同步到Redis中去，也是跟上面一样的思路，从Redis中获取到数据，然后修改，最后回填。

#### 4.删除购物车

删除购物车是当我们点击一行商品的删除按钮后触发的，这个时候我们就不需要在意其他的信息，直接就可以根据商品id删除整行商品的信息，无需关注数量之类。

未登录的时候，我们从Cookie中获取数据，根据key进行删除，然后回填。

```
HashMap<String, TbItem> resultMap = new HashMap<>();
String cookieValue = CookieUtils.getCookieValue(request, USIAN_CART, true);
resultMap = (HashMap<String, TbItem>) JsonUtils.jsonToMap(cookieValue, TbItem.class);
if (resultMap.containsKey(itemId)) {
    resultMap.remove(itemId);
}
CookieUtils.setCookie(request, response, USIAN_CART, JsonUtils.objectToJson(resultMap), true);
```

但是如果没有登录，我们就需要将数据同步到Redis中去，也是跟上面一样的思路，从Redis中获取到数据，然后修改，最后回填。