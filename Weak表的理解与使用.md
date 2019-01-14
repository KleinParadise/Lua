在lua中任何对内存块的引用都会使引用数加1，但有一个例外：weak table，它对内存块的引用不会使引用增1。
```lua
local a = {}
local key = {"123"}
a[key] = 1

key = {"234"}
a[key] = 2

collectgarbage()

for k,v in pairs(a) do
    print(k,v)
end
```
上述代码将{"123"}这个table的引用赋值给了key,同时将{"123"}这个table的引用也赋值给了a[key]此时对{"123"}的引用的数量为2.将table{"234"}的引用赋值给key,则key不在持有对{"123"}这个table的引用,其引用数量要减1.将{"234"}这个table的引用也赋值给了a[key]此时对{"234"}的引用的数量为2.运行collectgarbage()对内存进行回收,打印表a结果如下:
```lua
table: 0x7f9fe1e003e0	1
table: 0x7f9fe1e00750	2
[Finished in 0.0s]
```
表a中存有{"123"},{"234"}这两条数据,但是我们不通过遍历表a已无法访问到{"123"}这条数据。按照代码逻辑,{"123"}数据应该被GC回收。目前未被回收的原因是表a还保持对{"123"}的引用,因此无法回收。为了避免这种情况,我们可以通过lua weak表来实现对{"123"}数据的回收,代码如下:
```lua
a = {}; 
b = {};    
setmetatable(a,b); -- 设置a为weak table 
b.__mode = 'k'; 

key = {"123"};    -- 增加"{}"内存块的一个引用 - key，引用数1 
a[key] = 1; -- weak table引用不增引数，所以"{}"内存块的引数还为1 
key = {"234"}    -- 改变key指向新增的"{}"内存块，上面的"{}"内存块引数减一为0 
a[key] = 2     -- 如上上一样 

collectgarbage();   -- 调用GC，清掉weak表中没有引用的内存 

for k, v in pairs(a) do 
  print(v);    -- 只输出一个2 
end 
```

##### lua weak表的应用之一:记忆函数
一个函数的实现很复杂,每次执行都需要进行复杂的运算,我们往往采用空间换时间的方法来提高函数的运行效率。即将参数作key,将运行结果存入缓存table。当再次调用该
函数时,先看缓存table中有无结果,有就直接返回,无就经过函数计算并存入缓存table然后返回。示例代码如下:
```lua
local results = {}
function mem_loadstring (s)
    if results[s] then 
         return results[s]
    else
         local res = loadstring(s)
         results[s] = res
         return res
    end
end
```
如果该函数写在服务器,而服务器长期运行,很少停机维护。那么results这个table就会无限扩大,甚至撑爆内存。如果将results改为weak table。results中数据就会被定期回收,那么将不会增加服务器压力,使代码更加健壮。实现代码如下:
```lua
local results = {}
setmetatable(results, {__mode = "v"})
function mem_loadstring (s)
    if results[s] then 
         return results[s]
    else
         local res = loadstring(s)
         results[s] = res
         return res
    end
end
```

##### lua weak表的应用之二:关联对象属性
当我们需要给任一对象添加一个属性的时候，可以在外部单独做一弱key表，然后以对象为key值，属性值为value。这样即可以方便的访问这个属性，也不影响该对象的释放。而且对象本身没任何修改，能很好的保持对象本身的独立性。实例代码如下:
```lua
local tCacheWeakTab = {}
setmetatable(tCacheWeakTab, {__mode = "kv"})

local obj = {
    ["name"] = "张三",
    ["age"] = 18,
}

tCacheWeakTab[obj] = {
    ["job"] = "码农",
}

print(tCacheWeakTab[obj].job)

obj = nil

collectgarbage()

for k,v in pairs(tCacheWeakTab) do
    print(k,v)
end
```

##### lua weak表的应用之三:设置默认值的表
```lua
local defaults = {}
setmetatable(defaults, {_mode = "k"})

local mt = {__index = function(t) return defaults[t] end}

function setDefault(t, d)
    defaults[t] = d
    setmetatable(t, mt)
end
```
