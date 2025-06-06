# Stream流

- map:相当于对数据进行一个操作，可以自定义返回值等
- distinct:可以去除流中的相同元素，注意（*该方法依赖的Object的equals方法来判断是否是相同对象，所以要重写equals方法，否则只有对象地址一样时才会被认为是重复*）
- sorted:可以对流中的元素进行排序，传入空参时使用的是实体类的比较方法
- limit:设置流的最大长度，超出部分将被抛弃
- skip:跳过流中的前n个元素，返回剩下的元素
- **flatMap**:map能把一个对象转换成另外一个对象来作为流中的元素，而flatMap可以把一个对象转换成多个对象作为流中的元素
- 中间操作（filter,map,distinct,sorted,limit,skip,flatMap）
- 终结操作（forEach,collect,count,max,min,reduce归并,查找与匹配）
- forEach:遍历所有元素
- count:计算元素数量
- min&max:返回的是option对象，这里和sorted一样，得指定比较规则
- collect:把当前流转换成一个集合（list, set, map）
  - Collectors.toList()
  - Collectors.toSet()
  - Collectors.toMap(key, value)
- anyMatch:可以用来判断是否有任意符合匹配条件的元素，结果为boolean类型
- allMatch:可以用来判断是否都匹配条件，结果也是boolean类型，都符合则为true
- noneMatch:是否都不符合，都不符合则为true
- findAny:获取流中的任意一个元素，该方法无法保证获取的是流中的第一个元素，只是匹配到
- findFirst:获取流中的第一个元素
- reduce:对流中的数据按照你制定的计算方式计算出一个结果，并返回一个Optional描述归约值（如果有）