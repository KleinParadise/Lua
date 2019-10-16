```Lua
local _class={}
 
function class(super)
	local class_type={}
	class_type.ctor=false
	class_type.super=super
	class_type.new=function(...) 
			local obj={}
			do
				local create
				create = function(c,...)
					if c.super then
						create(c.super,...)
					end
					if c.ctor then
						c.ctor(obj,...)
					end
				end
 
				create(class_type,...)
			end
			setmetatable(obj,{ __index=_class[class_type] })
			return obj
		end
	local vtbl={}
	_class[class_type]=vtbl
 
	setmetatable(class_type,{__newindex=
		function(t,k,v)
			vtbl[k]=v
		end
	})
 
	if super then
		setmetatable(vtbl,{__index=
			function(t,k)
				local ret=_class[super][k]
				vtbl[k]=ret
				return ret
			end
		})
	end
 
	return class_type
end
```

使用示例

```lua
-- 基类
base_type=class()		-- 定义一个基类 base_type
 
function base_type:ctor(x)	-- 定义 base_type 的构造函数
	print("base_type ctor")
	self.x=x
end
 
function base_type:print_x()	-- 定义一个成员函数 base_type:print_x
	print(self.x)
end
 
function base_type:hello()	-- 定义另一个成员函数 base_type:hello
	print("hello base_type")
end

-- 子类
test=class(base_type)	-- 定义一个类 test 继承于 base_type
 
function test:ctor()	-- 定义 test 的构造函数
	print("test ctor")
end
 
function test:hello()	-- 重载 base_type:hello 为 test:hello
	print("hello test")
end

-- Test
a=test.new(1)	-- 输出两行，base_type ctor 和 test ctor 。这个对象被正确的构造了。
a:print_x()	-- 输出 1 ，这个是基类 base_type 中的成员函数。
a:hello()	-- 输出 hello test ，这个函数被重载了。
```


类的new函数，这里每定义一个class都会给它生成一个new函数，之后就可以通过className.new(…)来创建对象。
```lua
class_type.new=function(...) 
    local obj={}
    do
      local create
      create = function(c,...)
        if c.super then
          create(c.super,...)
        end
        if c.ctor then
          c.ctor(obj,...)
        end
      end

      create(class_type,...)
    end
    setmetatable(obj,{ __index=_class[class_type] })
    return obj
end
```
New函数里定义了一个递归函数create，该create判断传入的c（也即当前类）是否存在super（也即父类）如果存在则递归调用，这样一来就沿着类的递归链将类所有的父
类自上而下传入create函数。而之后则调用其ctor（也即构造函数，如果存在）对对象进行构造，重点在这里，这里调用构造函数传入的第一个参数self是obj，也就是
new函数第一句话申明出的局部对象。如此调用则在类（包含当前类及其所有父类，下同）的ctor中声明的变量通通都在obj中被创建，就成功的实现了将对象初始化
（当然这也意味着不在类ctor中声明的变量不会在obj中被创建）。且在类ctor申明的变量的创建延迟到了对象被创建时因而这部分变量也不会在类中，相对那些现在类中
创建变量之后实例化时把类中所有变量拷贝一份到实例化对象中的方法而言避免了一定的无意义内存开销。

函数的结尾是一句：
```lua
setmetatable(obj,{__index = _class[class_type] })
```
```lua
	local vtbl={}
	_class[class_type]=vtbl
 
	setmetatable(class_type,{__newindex=
		function(t,k,v)
			vtbl[k]=v
		end
	})
```
这里看到_class[class_type]其实指向了一个表vtbl（名字的命名来源于c++类对象中的虚表），该vtbl被设置为class_type的元表且对class_type的__newindex操作
被hook到了在vtbl上进行。
这里逻辑初步看起来不知其所以然，其真实目的是这样一做，之后在类（也就是这里即将被返回的class_type表）中添加的任何属性（变量）或方法（函数）都被实际在
vtbl中创建，这时候回过头看看.new方法中的那句就瞬间明白了——这样一来就可以通过对象来访问到类中的方法了（当然也包括那些不在类的ctor中被申明的变量）。
因此示例中test类在被class(base_type)创建出后添加的hello()方法，就能通过对象a:hello()来访问到。

```lua
	if super then
		setmetatable(vtbl,{__index=
			function(t,k)
				local ret=_class[super][k]
				vtbl[k]=ret
				return ret
			end
		})
	end
```
这里是用来实现类的继承逻辑的，test类继承自base_type类，test中的vtbl只保证了通过对象a能够访问到test中添加的方法，但是对于那些在test的父类base_type中
的方法（比如例子中的print_x()）就得靠这里来访问。这里给vtbl再设置了一个元表，其中__index原方法指向的就是父类的vtbl（这里保存有父类中的方法），因此最
终的对象访问一个方法（比如print_x()），在其直接类（比如test）的vtbl中找不到时会向上到类的父类的vtbl中找，并如此递进直到找到了或者确定不存在为止。
Vtbl[k]= ret这句是在第一次在父类中查找时把查找结果拷贝到当前类，从而避免了下一次访问的重复查找。
对于那些不在类的ctor()函数中申明的变量因为会保存在类的vtbl表中，该表对类唯一因而为类所有对象所共有。因此这种变量的性质有点类似c++中的静态变量。

