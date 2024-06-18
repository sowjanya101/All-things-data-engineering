# configparser

    config.ini and config.conf are the conf fies used commonly where data is stored in sections of key value pairs. The strings varaibles in here needn't be enclosed in the quotes
    # Reading "C:\Users\sowja\OneDrive\Desktop\work\DE_notes\config.ini" and 
    # Reading "C:\Users\sowja\OneDrive\Desktop\work\DE_notes\config.conf"

    import configparser
    config = configparser.ConfigParser()
    config.read('C:\\Users\sowja\OneDrive\Desktop\work\DE_notes\config.conf')

    # using items 
    for k,v in config.items('default'):
        print(k, ': ', v)
        
    # using get method (section, key)
    config.get('default', 'name')

    # using section as arg, and providing a default value 
    config['default'].get('lastname', 'j')


# sort, sorted 
- sort sorts a list inplace
- sorted returns a sorted list by taking any iterable - list, string, dict, tuple 
- sorted also accepts a key, that could be a func, where we can define the order 

        # sorting a str
        sorted('cab') ==> ['a', 'b', 'c']

        # sorting a dict
        d = {'a': 3, 'c':1, 'd': 4}
        sorted(d, key=lambda x: d[x])
        Res: ['c', 'a', 'd']

        # sorting without key 
        d = {'a': 3, 'c':1, 'd': 4}
        sorted(d)

        # sorting a list based on another lists order 
        lst2 = [3, 2, 5, 1]
        lst1 = [1, 2, 3, 5]

        pos_d = {}
        for i, item in enumerate(lst2):
            pos_d[item]=i

        sorted(lst1) ==> [1, 2, 3, 5]
        sorted(lst1, key=lambda x: pos_d[x])  ==> [3, 2, 5, 1]


# create a package and make it available for all other python functions


# OOPS
## class variables and instance variables 
        class Test:
            name = 'sowji'
            
            def __init__(self, second_name):
                self.second_name = second_name

        test_obj = Test('J')
        print(test_obj.name) ==> accessing class vars
        print(test_obj.second_name)  ==> accessing instance vars

- params of class instance(the __init__ methods parameters) are passed while instantiating the class 
- parmas of the methods are passed when methods are called on class obj 


        class Polygon:
            sides = 0
            def __init__(self, sides):
                self.sides = sides 

            def area(self, length):
                return length*length

        pol_obj = Polygon(4)
        area = pol_obj.area(5)

        # access class var sides without creating an obj out of Polygon class 
        Polygon.sides ==> 0

## inheritance 
- inherit/overwrite the class attributes/methods of a parent class in a child class

# child class - inherting Polygons init and area methods
        class Square(Polygon):
            pass

        sqr_obj = Square(4)  
        print(sqr_obj.sides)    --> 4
        print(sqr_obj.area(5))  --> 25


## child class - inherting init method, overwriting area method
        class Rectangle(Polygon):
            def area(self, length, width):
                return length*width
            
        rec_obj = Rectangle(4)
        print(rec_obj.sides)

        print(rec_obj.area(4, 5))

        4
        20

## child class- overwriting the init method
        class Rectangle(Polygon):
                def __init__(self, perimeter):
                    self.perimeter = perimeter

            
        rec_obj = Rectangle(10)
        print(rec_obj.perimeter)  --> 10
        print(rec_obj.sides)  # throws error


## child class- adding to the init method of parent class, so that we can have
## instance vars of both child and parent class

        class Rectangle(Polygon):
                def __init__(self, sides, perimeter):
                    super().__init__(sides) # super enables the instantiation of parent class as well
                    self.perimeter = perimeter
                    
        rec_obj = Rectangle(4, 10)
        print(rec_obj.perimeter)
        print(rec_obj.sides)  

        10 
        4 


## multiple inheritance 
        class Human:
            def __init__(self, name, age):
                self.name = name 
                self.age = age
                
        class Dancer:
            def __init__(self, style):
                self.style = style
            
        # super().__init__() by default provides the class instance, we dont need to pass self
        # explicitly again
        # when using multiple inheritance, an init of a parent class needs to be instantiated
        # by using the class name directly, and self needs to be passed explicitly 
        class student(Human, Dancer):
            def __init__(self, name, age, style):
                Human.__init__(self, name, age)
                Dancer.__init__(self, style)

        s1 = student('sowj', 27, 'Hip')
        print(s1.name)
        print(s1.age)
        print(s1.style)
        sowj
        27
        Hip

        class student1(Human, Dancer):
                pass
        s1 = student1('sowj', 27)
        print(s1.name)
        print(s1.age)
        sowj
        27

        s1 = student1('Hop') -- error

        # by default, if init method is not defined in child class, the init method of
        # first parent class inherited is instantiated, and the args are accepted accordingy 
        # while instantiating the child class
        #  student1('Hop') - throws an error


## operator overloading 
### change the behaviour of opeartors like +,-,< etc using special methods in class
        class Book:
            
            def __init__(self, price):
                self.price=price
                
        b1 = Book(10)
        b2 = Book(30)

        # if we try to take price diff between b1 and b2 
        print(b2.price-b1.price)

        # the above works, but what if we want to do that directly using classes
        # b2-b1 throws error, because the - accepts two ints and subtracts them, 
        # to sub two classes, we have to define that as a special method in class
        class Book:
            
            def __init__(self, price):
                self.price=price
                
            def __sub__(self, objtwo):
                return self.price-objtwo.price
        b1 = Book(10)
        b2 = Book(30)
            
        print(b2-b1)

- a+b is internally represented as a.__add__(b)
- __getitem__: obj[key] is same as obj.__getitem__(key). obj[key] usually returns an item at key index of a list, a value at key in a dict. The same can be defined in a function and override the behaviour
- __setitem__: obj[key] = value is same as obj.__setitem__(key, value). Either assigns a value at index "key" in list or assigns value at "key" in dict 





# split 
* split() - by default splits by "whitespace", ignores more than one space, leading and trailing spaces
* split('{delim}') - gives a list sep by delim along with the delimiters. if there is nothing between two limiters, it will show ''. Doesn't ignore more than one delimiters in sequence

        s = ' hi tina  huang   '
        # ignores more than one space, leading and trailing spaces
        s.split() => ['hi', 'tina', 'huang']

        # split by delimiter gives blank strings in the list if there is nothing between two delimiters
        s.split(' ') => ['', 'hi', 'tina', '', 'huang', '', '', '']


        s = ',hi, hello'
        s.split(',')  => ['', 'hi', ' hello']

        # Doesn't ignore more than one delimiters in sequence, and gives a blank string in the output list
        s = 'hi,,hello'
        s.split(',')
        ['hi', '', 'hello']




# regex 
## re.match

    - matches the patter with the str and returns the groups if asked for. The groups needed should be specified using "()" in the pattern.
    - group(0): gives entire matching string 
    - group(1): gives first group enclosed in "()" the pattern 
    - group(n): gives nth group enclosed in "()" pattern 
    - re.search works similar to re.match, but - 
      - re.search searches are returns the match anywhere in the str
      - re.match searches for the pattern from the beginning of the str 
  
        import re
        re.match(pattern, str, flag) => flag cold be an re.IGNORECASE etc

        Ex: 
        Given s = "Toy Story (1995)" 
        get the movie name and year seperated 

        import re
        pattern = r'([a-z ]+)(\d+)'
        re.match(pattern, 'toy story 1996').group(1) 
        => 'toy story ' 

        re.match(pattern, 'toy story 1996').group(2)
        => 1996

        re.match(pattern, 'toy story 1996').group(0) 
        => 'toy story 1996'

        - enclose only whats needed to be extracted with groups() method using "()"


        # re.search can directly look for year in the given string without having to match the entire pattern from the beginning 

        re.search(r'(\d+)', 'toy story 1996').group(1) => 1996 
        re.search(r'([a-z ]+)', 'toy story 1996').group(1) => 'toy story '


        # multiple matches using finditer 
        string = 'the number: 99600, you can all anytime, another is 98766'
        for i in re.finditer(r'([0-9]+)', string):
            print(i.group())

            99600
            9876    

# RAM 
- computers are designed to bring ele at an address using O(1) - random access. The mem address 8978374 can be accessed in same time as that of address 123
- the way lists or any arrays are able to grab the item at a particular index is, the array var knows the starting add of the array. if the array had only numbers, (numbers usually take 2 bytes). hence the pointer would go from the starting mem add + (byte size)*index
- for array of strings, there is no way to tell the size each ele of the array would take, hence arrays use "referential arrays", where the original array at each index would stores the add of the string thats supposed to be in that index
- shallow copy: 
    - a = [1, 2, 3]
    - b = a
    - instead of creating a new array, the b would point to the mem add of 1, 2, 3
    - this is fine since 1, 2, 3 are ints, and ints are immutable. hence doing b[0] = 5, creates 5 at a new address and the 0th index of b points to that new add
    - but this would change if the data store in array a is mutable like list
    - if b is modified a would also show changed values

            a = [[1,2], 3, 4]
            b = a
            b => [[1, 2], 3, 4]
            b => b[0].append('new')
            b => [[1, 2, 'new'], 3, 4]
            a => [[1, 2, 'new'], 3, 4]

    - **python if shallow copied is also changing the data even if the items in the source list are immutable, so make sure to always deep copy 
    - b[:] = a => works as deep copy 

# MISC 
## modify list in-place
- whenever a list is asked to be modified inplace, have a sep index that tracks modification from the beginning while the other pointers move around the array.
- when we write a file to the disk, it stores the unicode ASCII (text) chars as serialized bytes (probably encoded with utf-8 codec) automatically 


# list 
## amoritization 
[LINK HERE] https://www.youtube.com/watch?v=WIuITZgGYG8&t=526s

- lists need to be contiguous in memory for them to have access complexity of O(1)
- since we wouldn't know the capacity of the list beforehand, it is diffcult to give that contiguous memory to the list 
- lists are designed to take amoritized capacity when they are built 
- when a list is created, it would be created w a capacity of 1, but when an ele is added, list aquires double the capacity of memory somewhere else, and copies the existing elements over to the new location
- from here onwards, if another ele is added, it can be added w o(1) complexity, as the additional capacity is aquired in the above step.
- Now, if the size of the list reaches its capacity, list has to again aquire double the memory capacity and copy over the existing elements over to the new location, which means in some cases, inserting an ele at the end of an array is o(n) and sometimes(when the size of array is less than the capacity of the aray) its o(1). When calculated for inserting n elements, the complexity comes out to be o(1)
- But in amoritized terms, an append operation is said to have a complexity of o(1) amoritized
- generally a list obj has a size, capacity and a pointer to the memory add in the background


# map function 
- map(func, iter)
- applies func to each iterable and returns the result
- we can either iterate through the map results or convert it into list/set 

            Ex:1
            lst = [1,2,3]
            list(map(lambda x:x+x,  lst))
            -- [2, 4, 6]

            Ex:2 
            def func(x):
                return x**2
            for i in map(func, lst):
                print(i)

            Ex3: 
            lst2 = [1,2,3]
            map(lambda x,y: x+y, lst, lst2)

# filter 
- filter(func_to_return_bool, iter)
- func_to_return_bool - this could be lambda, and it should return either true or false and filter filters the true records 
- need to iterate through the filter res or use list() or set() methods 
- similar to map, but gets filtered source iter based on the filter cond
        list(filter(lambda x: x%2==0, [1,2,4]))

# lambda 
- is an anonymous func 
- if assigned to a var, the var needs to be called 
    a = lambda x: x+1
    a(1) -> 2

    res = [lambda x: x+2 for i in range(5)]
    <!-- the res is not directly executed, we get a list of lambda funcs that needs to be called -->
    for i in res:
        print(i())

- needs a method like map that can apply the lambda on an iterable 


# removing ele from dict and set 
- pop for set and popitem for dict. Removes ele/pair arbitrarily and returns the removed ele/pair. The org data struct is modified
- similar to list, but list.pop() by default removes ele at -1 position
- Additionally,dict has pop({key}) method, which lets us specify the key to delete from the dict, and this returns the value thats deleted
- for deleting a specific ele in a set, it has remove(ele) method. Doesnt return anything, just modifies original set in-place.
  

# xor 
if the elements are the same:
1 1 -> results in 0
0 0 -> results in 0
if one of the ele is 1: 
1 0 -> results in 1
0 1 -> results in 1
- a^b (xor symbol)

This was use**d** for identified ele thats deleted from original arr wth constant space (not using maps)
arr1 = [5, 5, 7, 7]
arr2 = [5, 7, 7]

res = 0
for ele in arr1+arr2:
    res = res^ele

0 ^ 5 - 5
5 ^ 5 - 0
0 ^ 7 - 7 
7 ^ 7 - 0 
0 ^ 5 - 5 
5 ^ 7 - 2
2 ^ 7 - 5
Thsi works as every ele has a pair that cancells each other and returns 0, except one which when xor'ed with 0 would return that one pairless ele

# zip 
zip of two arrays, creates pairs of elements from each arr according to their indexes, and the length of zip is equal to the smallest arrays length. The other remaining eles from the large arr are discarded.

        list(zip([1,2,3], [4,5]))
        [(1, 4), (2, 5)]


# generator 
        def generator_func(max):
            initial_val = 0
            while initial_val < max:
                yield initial_val  # changes a func to generator
                # inc initial val
                initial_val += 1

        gen = generator_func(5)
        print(next(gen)) - 0
        print(next(gen)) - 1
        print(next(gen)) - 2
        print(next(gen)) - 3
        print(next(gen)) - 4
        print(next(gen)) # error -StopIteration


# defaultdict
- dict can accept any immutable datatype as its key, it can accept tuple as well, but not dict

        from collections import defaultdict
        # defaultdict(default_func)
        d = defaultdict(list)
        for i in range(5):
            d[i] = i
        d[6] => []

        lst = [1,2,3]
        d_int = defaultdict(int)
        for ele in lst:
            d[ele] = 1 
        d_int[4] => 0

        d3 = defaultdict(lambda: 'no value')
        d3['a'] => 'no value'


# module
- any python file that can imported into another .py file 
- if a folder has __init__.py and has bunch of .py files, its a package 

script.py 
main_package/
    __init__.py
    main.py

    sub_package/
        __init__.py
        sub.py

main_package is a package and hence we can use "." notations to import these modules inside the package into script.py 

Inside script.py 
from main_package.main import my_func
from main_package.sub_package import sub


# datetime 
- import datetime as dt
- dt.time(hour, min, sec) -> time obj
- dt.datetime(year, month, day, hour=0, minute=0, second=0) -> datetime obj 
- dt.date(year, month, day)
- we can use these in comparisions



# pandas 

## series 
- series is a column in pandas df 
- it has benefits of both list and a dict and supports more complex methods than list and dict 
- it is ordered like a list, and it can be associated with a key(index) like a dict 
- if index is not provided, the default values are 0, 1, 2...

        # creating a series 
        lottery_nums = [4, 6, 13, 21]
        pd.Series(lottery_nums)

        0     4
        1     6
        2    13
        3    21
        dtype: int64


        # creating a pandas series using dict 
        sushi = {
            'salmon': 'orange', 
            'Eel':'brown', 
            'tuna': 'red'
        }
        pd.Series(sushi)

        salmon    orange
        Eel        brown
        tuna         red
        dtype: object

- even though python dict isn't ordered, pandas series maintains the order. the key is taken as index, and the value as col


## methods on series
- Series is a class, and we have a lot of methods available on this class

        lottery_nums = pd.Series(lottery_nums)
        # we can call methods like sum, mean, product on this 
        lottery_nums.sum() --> 44
        lottery_nums.product()


## attributes of series 
- since Series is a class, along with the methods it also has attributes
- lotter_nums.size  --col len
- lottery_nums.values -- values of the col 
- lottery_nums.index
- Also, the Series values are made up of numpy array class, where as the index is made up of pandas class

## parameter vs argument 
- for any python function, method, argument.. the variable in the function/others definition is called paramter and the actual value we pass to that is called argument

            def func(a):
                pass 

            func(a=10) # a is the paramter and 10 is the argument


## read csv 

    names = pd.read_csv(path, usecols=['Name']) # usecols-specify a list of cols to fecth from csv file
    # names is still a df. To convert it into series, we can use squeeze method
    names.squeeze("columns") # this only works if df has one column
    # columns is used as an arg to specify to squeeze col wise

    # converting one col as index while reading csv 
    names = pd.read_csv(path, usecols=['Name', 'age'], index_col='Name')
    # earlier, there was a default index (0, 1, ), now the name becomes index
    # After name is made an index, the df contains only one col, and can be converted to a series 
    names = pd.read_csv(path, usecols=['Name', 'age'], index_col='Name').squeeze('columns')



## passing pandas series to python functions 
- series can be passed to python functions like below 
- len(series_obj) = returns length
- list() = converts to list 
- dict() = converts to dict 
- max(), min(), type()
- sorted(series_obj) = returns a sorted list

## inclusion keyword
- can use in with series. By default, it looks for the value in index, for values, use series.values()
- 0 in sushi => looks in index
- 0 in sushi.values() => looks in col
- 'salmon' in sushi_series => True

## sort_values, sort_index
- these are used to sort values of the series/index of the series. If values are sorted, index is rearranged automatically and vice versa
- returns a new series
- sushi = sushi.sort_index(), sushi = sushi.sort_values()

## accessing series with index location
- series.iloc[0], series.iloc[:5], series.iloc[[2, 5]]
- supports all list slicing operations
- for multiple indices, can specify them in a list
- specifying a list of indexes returns a series object


## accessing series with index label
- sushi.loc['tuna'], sushi.loc[['salmon', 'tuna']]
- select multiple labels using a list
- even though the series has lables, index is still available in the background and ele of series can be accessed using iloc
- sushi_series.iloc[2]
- specifying a list of labels returns a series object

<!-- Note:  -->
- when sort_values is performed on a series, the index is moved corresponding to the sorted values. Meaning, index is associated to the value even if its sorted, and vice versa when sort_index is applied

            
            lottery_series
            0     4
            1     6
            2    13
            3    21
            dtype: int64

            lottery_series.sort_values(ascending=False)
            3    21
            2    13
            1     6
            0     4
            dtype: int64

- But, when there are index labels, and its sorted, while the labels associated with values doesn't change if the sort order changes, the hidden numerical index that lets us do .iloc on labelled series remains the same from 0 onwards even the sort_values, sort_index is applied
- Below two may result in diff values
- sushi_series.sort_values().iloc[0]
- sushi_series.iloc[0]
- but accessing a value with label in labelled series, and with index in indexed series never change. This diff is only for labelled series accessed with iloc


## get method
- lets us provide a default value if the label doesn't exist
- sushi.get('panchow', 'nonexistent')
- when a list of labels is asked for, and one of them is not present in the series, an error is thrown
- sushi.loc[['panchow', 'salmon']] -> errors
- sushi.get(['panchow', 'salmon'], 'one of the eles in list is not present') -> outputs "'one of the eles in list is not present'"

## edit the series 
- select the index/list of indexes to edit or a label/series of labels to edit and assign new value of list of new values

        sushi_series.loc[['salmon', 'tuna']] = ['lighter orange', 'lighter red']
        sushi_series

        lottery_series.iloc[0] = 10
        lottery_series


## copy method
- creating a series out of a df, using squeeze method is a shallow copy, meaning changes to the series, changes the df 
- to create a deep copy, make use of copy method


            df = pd.read_csv('path', index_col='Name')
            series_obj = df.squeeze('columns')
            # series_obj is now a shallow copy

            series_obj = df.squeeze('columns').copy()
            # series_obj is now a sep obj (deep copy happens)

## broadcast 
- apply an operation to every ele of a series 
- like adding a const/subtracting/multiplying a constant 
- can use eitehr methods or mathematical operators 
- lottery_nums + 10 <=> lottery_nums.add(10)   - adds 10 to every ele
- lottery_nums * 1.25 <=> lottery_nums.mult(1.25)
- .div(), .sub()

## apply 
- series.apply(func). Applies func to every ele of a series 
- lottery_series.apply(lambda x: x+10). Adds 10 to every ele

## map 
- series.map(dict/series). Looks up each value in series in the dict/series's key/label and brings corresponding value 
- if a value is not present in the series, return Nan(FloatType)
- value to the label of map_obj, and brings the value of map_obj

        map_obj = {
            10: 'ten', 
            6: 'six'
        }
        lottery_series.map(map_obj)

        <!-- lottery_series -->
        0    10
        1     6
        2    20
        3    30
        dtype: int64

        <!-- result after map -->
        0    ten
        1    six
        2    NaN
        3    NaN
        dtype: object



## pandas dataframe 
- two dim array, for series only row index is availabe, but for df, both row and col index are available
- represented as list of lists. Each inner list is a row

# sum method on series vs dataframe 
<!-- series.sum() vs df.sum() -->
- sum() takes axis parametere, which says along which direction it has to traverse and take sum 
- axis = index/0 >>traverse across each index and sum up the values
- axis = columns/1 >>traverse across each col and sum up the values 

        data = [
            [100, 200, 300], 
            [400, 500, 600]
        ]
        df = pd.DataFrame(data, columns=['ny', 'ca', 'texas'])
            ny	ca	texas
        0	100	200	300
        1	400	500	600

        df.sum(axis='index')/df.sum(axis=0)
        ny       500
        ca       700
        texas    900
        dtype: int64

        df.sum(axis='columns')/df.sum(axis=1)
        0     600
        1    1500
        dtype: int64


## selecting cols from a df

### single col
- df.column_name, df['column_name']  => series object 
- s = df.ny
- s is a series that is view/shallow copy of df, hence changes to the view also changes the df
- Ex: s.iloc[0] = 500 also changes the value of df at index 0 in ny column
- to avoid this, we can use copy() method like below
- s = df.ny.copy()/ df['ny].copy()

### multiple cols 
- df[['ny', 'ca', 'texas']] = returns a df 
- simialr to selecting indexes with loc and iloc, if we want multiple indexes, we provide them in a list and the result will be a series
- here the result will be a df 
- Unlike extracting one column, which creates a view, extracting multiple column creates a copy 


## adding col to the df 
- df['new_col'] = s + 10
- df['new_col'] = df['old_col'] * 10
- df['new_col'] = 'static value'
- All these adds the new col to the end of the df 
- to add it to a specic loc, use, df.insert(loc, column,value)
- df.insert(loc=1, column='new_col',value='static value')
- the loc parameter is the postiton amongst the cols to add the new col to 
- the new_col is created at 1st pos, after 'ny'

## dropna, fillna
df.dropna(how="any/all", subset=[])
- df.fillna(value='')
- fillna replaces all "nan" values with value provided
- to avoid this, select the column(series) and then assign it back to the df 
- df['salary'] = df['salary'].fillna(0)
- df['Name'] = df['Name'].fillna('unknown')
- Note: the substet parameter is not availbale in the fillna method


## astype
- df['column'] = df['column'].astype(int)
- if the column is of float type with NaN values, casting it to int doesnt work. First replce them and then cast 
- df['column'] = df['column'].fillna(0).astype(int)
- df.dtypes -> prvides data types
- "category" is a special data type that we can cast a col to, to reduce the storage footprint. This is used when a col has only a few unique values 
- df['column'] = df['column'].astype(category)

## value_counts and nunique
- df['column'].value_counts() => count of each value in a column
- - df['column'].value_counts(normalize=True)*100 => percentage count of each value in a column
- df['column'].nunique() => unqiue count of a column

## df.info()
- info about the df, each col its not null count, total df size etc
- this is great to find if a col has missing values, and apply required transformations.

## effecting multiple columns of a df 
- df[['ny', 'ca']] = df[['ny', 'ca']].astype(int)


## sort_values, sort_index in df
- df = df.sort_values('col', ascending=True, na_position="last") -- last is default, "first" is other value 
- ascending is true by default. The index is changed accordingly
- df = df.sort_values(['col1', 'col2'], ascending=[True, False])
- first sorts by col1 and then col2. 
- df = df.sort_values(['col1', 'col2'], ascending=False)
- sorts both cols in descending 

- to sort index of the df 
- df.sort_index(ascending=True). True is default

## rank
- gives rank on a column 
- df['rank'] = df['salary'].rank(ascending=True)
- gives same rank to the same values, and skips ranks

## pd.to_datetime
- pd.to_datetime takes a series as an arg and optional format 
- df['start_date'] = pd.to_datetime(df['start_date'], format='%y-%m-%d')
- if there is a time column hh:mm AM/PM 
  - 12:03 AM 
  - 11:5 PM 
  - use, pd.to_datetime(df['start_time'], format='%H:%M %p')
  - this produces datetime in the format YYYY-mm-dd hh:mm:ss
  - to only select the time part, use
  - pd.to_datetime(df['start_time'], format='%H:%M %p').dt.time

## filter a df 
- pandas df needs a series of booleans to filter, returns rows that are True in the series 
- ex: filter row  where gender=Male
- df['Gender'] == 'Male' : This returns a series of booleans(broadcast)
- use this in the df 
- df[df['Gender'] == 'Male'] --> returns filtered df 
- df[df['start date'] > '2021-12-31']
- import datetime as dt  
- df[df['start time'] > dt.time(12, 0, 0)]
- we can also perform series == series  series > series 
- For more than one cond

    is_finance = df['department'] == 'Finance'
    is_female = df['gender'] == 'Female'

    df[is_finance & is_female]
    df[is_finance | is_female]

- Along with math operators on series, we can also use "isin". isin method takes a collection obj, like list, tuple, series
- df['gender'].isin(['F', 'Female'])


## isnull, notnull methods
- df['col'].isnull() -> returns a boolean series
- df['col'].notnull() -> returns a boolean series 

## between method 
- available on series .between(lower, upper). lower and upper are inclusive 

## duplicated method 
- works on series and df
- df['col'].duplicated(keep=), df.duplicated() --> checks for dups across all cols
- keep: 'first', 'last', False
- returns a boolean series, 
  - if a value in series is duplicated 
       - returns False for first occurance and True for rest of the duplicated occurances if keep='first'
       - returns False for last occurance and True for rest of the duplicated occurances if keep='first'
      - returns True for all the duplicated occurances if keep=False
- can filter the df, give rows where name is unqiue
  - df[~df['Name'].duplicated(keep=False)]

## drop_duplicates method
- works on both df and series
- df.drop_duplicates(keep="first/last/False")
- if no cols are specified works on whole df
- can specify one col or list of cols
- df.drop_duplicates('col1', keep="first/last/False")
- df.drop_duplicates(['col1', 'col2'], keep="first/last/False")




# DSA 
### Bucket sort 
- problem: a dict holds a value and the number of time it occurs, get topk frequent elements from that dict with o(n)
- if we take all values, sort them in desc order and then pick first k elements, and get the keys from dict 
- here sorting is nlogn 
- to make this o(n), create a list of lists of n size (bucket)
- put the elements that occurred i items in ith index
- ex: if 3 and 4 occurred 2 times, put [3,4] in 2nd index of the list(bucket)
- now, iterating frmo the end, pick k elements you need 
ex: 
nums = [1, 2, 1, 3, 1, 2], k = 2
- here 1 and 3 are most frequent, occurring 3 times,  since we need 2 (k=2) most freq elements, we pick 1,3 from below
buck = [[], [3], [], [1,3], []]
- the bucket should have 5 empty lists for 5 positions, so that 5th position can hold the number, if its repeated 5 times.
- this solution takes additional o(n) space
- another variation is to use freq array, where the index of this freq array is an actual ele of arr1 and the value is the freq of that ele arr1

            arr1 = [2, 3, 5, 5, 2, 2]
            since max is 5, we need an array with 5 positions, so array size of 6

            freq_arr = [3, _, 1, _, 2, _]
            Now, I can reference the index of this array to get freq of each ele. we can use this when solving relative order of one array with another


### use set instead of list 
- when you wanna traverse the list for each ele in the list
- using set makes it constant time, where as using list makes in o(n^2)

    ex: 
    for ele in list: # o(n)
        search = ele
        while search+1 in list: # o(n)
            search+=1 
            count+=1

    set = set(list)
    for ele in list: # o(n)
        search = ele
        while search+1 in set: # o(1)
            search+=1 
            count+=1    


## sliding window 
- use a window to determine longest subarrays
- for searching a window of certain length, use window+set
- typically, use two pointers to denote a window, and keep adding and removing the elemenst based on the window size


# asyncio 
- create a co-routine using a asyncio.create_task(func())
  - the function func should be defined like below 
    
    async def func():
        print('start')
        await asyncio.sleep(2)  
        print('end') 

    async def main():

        task1 = asyncio.create_task(func()) 
        
        # creates the task in event loop, so that it can be paused if its not doing anything 

        # the line "await asyncio.sleep(2)" is waiting for 2 seconds, while this happens, the event loop can pass the control to another task, so that it can start executing, making the program asynchronous

    asyncio.run(main()) -> this is how we start a async program 
    # if we need to run another function concurrently, while func() is waiting for something, we can do so by creating another task in the event loop 


        async def func(): #step3
            print('start')
            await asyncio.sleep(2)   #step4
            print('end') 
            return {'done': 'yes'}

        async def func1(): #step6
            for i in range(10): 
                print(i)
                await asyncio.sleep(0.25)  #step7...

        async def main():
            task = asyncio.create_task(func())  #step2
            task1 = asyncio.create_task(func1()) #step5

        asyncio.run(main())  #step1

        - once the step3 starts and the program encounters a await, it looks to see if there are any other tasks in the event loop that it can execute. 
        - there is, and its step5, so it starts the func1
        - and again in step7, it encounters await and checks if it can execute any other task, it doesnt find any as we are waiting till complete 2 seconds are done and the step7 has only 0.25 seconds
        - so every 0.25 seconds, it checks if it can execute any other task while it waits
        - now, if we wanna capture the result of func(), we need to again using await at the task level, like below 
      

        async def func(): #step3
            print('start')
            await asyncio.sleep(2)   #step4
            print('end') 
            return {'done': 'yes'}

        async def func1(): #step6
            for i in range(10): 
                print(i)
                await asyncio.sleep(0.25)  #step7...

        async def main():
            task = asyncio.create_task(func())  #step2
            task1 = asyncio.create_task(func1()) #step5
            res = await task  -> this waits until the func() is completely done and captures the result
            print(res)
            await task1

        asyncio.run(main())  #step1

## coroutine
- using async before a func name creates a coroutine object 
- if we call that func, like main() => returns a coroutine object 
- but it doesny execute it. For us to execute the coroutine obj, we have to use asyncio.run and this starts the event loop --> this is for main event loop
- for other coroutines, inside the event task, like func and func1
- these also doesnt start until we await it, or wrap it in a task
- when we await a coroutine, the other tasks pause until this is complete
  - await func()
  - print('hello') - this is not printed until the func() is executed

## task
- tasks help manage the multiple coroutines
- the idea of async program is to keep CPU occupied. If a co-routine is waiting on something, like if a coroutine has sent an API call and waiting on the result, now the cpu is idle.
- this is where a coroutine can help run another task while one coroutine is waiting for something else, thats not related to the current CPU 
- asyncio.start_task(func()#coroutine object) 
  
## await 
- await executes the coroutines
- whenever, await is encountered, it searches for other tasks in the event loop to execute, once everything is done and res is returned, then it proceeds to other steps 
- hence creating tasks and then awaiting them makes things run concurrently 

            task1 = asyncio.create_task(fetch_data(2, 1))
            res1 = await task1
            print('result: ', res1)
            
            task2 = asyncio.create_task(fetch_data(1, 2))
            res2 = await task2
                
- task1 is created in the event loop and in the next step, it is awaited, which means the func fetch_data is called. Now, inside the fecth data we encounter await asyncio.sleep(), since await is encountered and we are waiting out the sleep time, the program looks for anther task in the event loop. There are none, until after task1 is completed. Hence its finishes it and then does the same for task2
- Now, for task1 and task2 to execute concurrently, we need to create these two tasks in the event loop first and then await them

            task1 = asyncio.create_task(fetch_data(2, 1))         
            task2 = asyncio.create_task(fetch_data(1, 2))

            res1 = await task1
            print('result: ', res1)
            res2 = await task2
            print('result: ', res2)

- when await task1 hits, it calls the fetch_data and as soon as it encounters sleep, it looks for other tasks to start, we have task2, so it starts as well
- even though the print statement is available right after, they wait until all the tasks in the event loop complete 

## gather
- creates mutiple tasks at once, and awaited gather returns the results of all the coroutines in a list 
- res = await asyncio.gather(fetch_data(2, 1), fetch_data(1, 2))
- but this doesnt stop if one of the coroutines were to fail, poor error handling
- to help with this, a context manager is available as "TaskGroup"


        async with asyncio.TaskGroup() as tg:
        res1 = tg.create_task(fetch_data(5, 1))
        res2 = tg.create_task(fetch_data(1, 2))

        print('result: ', res1.result())
        print('result: ', res2.result())


# makefile 
- Makefile is used as a tool to perform multiple repeated operations, whenever a code base is changed like
  * installing requirements
  * creating distributions and binary distributions
  * running test cases
  * running code 
  
- we define all the above steps in a Makefile(with no extension) with label and recipe like below 
    Inside Makefile
    run: 
        python -m package 
    install: 
        python -m pip install -r requirements.txt
    build: 
        se
- Makefile is created inside the project folder at the package level 
- Makefile works out of the box for linux OS, but windows, download make utility using chocolatey package manager. This allows us to use "make" in powershell
- to execute the steps defined in Makefile, use it as below in powershell 
  - make install 
  - make run 
  - make build etc 
- there are other things like checking for pre-conditions, creating makefile commands to support mutiple OS, running all commands in sequence etc.


# building binary distribution (wheel) and source distribution(tar.gz) of your package 
- these are used to package code and share across multiple machines, upload to a repo like pypi. This makes installing the module/package in a new machine simpler
- sometimes, people provide a .whl file instead of publishing it to pypi repo, where we usually install the packages from 
- in this case, download the wheel file and use "pip install file.whl" and this is as good as any other package installed from pypi, and the modules in the code can be imported into any other code 
- this is not only used for packaging modules that can be used in another python modules, it can also be used for running a package as an application on command line 
[Link here] https://www.youtube.com/watch?v=n2d_7RPTKlk 

    
# ord, chr
- conver ascii character to decimal representation and vice versa
- ord('A') = 65
- ch(65) = 'A'

