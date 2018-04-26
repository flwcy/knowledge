### JS练习题答案

> from https://www.codewars.com

1. Solution:

   ```javascript
   function likes(names) {
     return {
       0:'no one likes this',
       1:names[0]+' likes this',
       2:names[0]+' and '+names[1]+' like this',
       3:names[0]+', '+names[1]+' and '+names[2]+' like this',
       4:names[0]+', '+names[1]+' and '+(names.length - 2)+' others like this'
     }[Math.min(4,names.length)];
   }
   ```

2. Sample Tests:

   ```java
   describe('example tests', function() {
     it('should return correct text', function() {
       Test.assertEquals(likes([]), 'no one likes this');
       Test.assertEquals(likes(['Peter']), 'Peter likes this');
       Test.assertEquals(likes(['Jacob', 'Alex']), 'Jacob and Alex like this');
       Test.assertEquals(likes(['Max', 'John', 'Mark']), 'Max, John and Mark like this');
       Test.assertEquals(likes(['Alex', 'Jacob', 'Mark', 'Max']), 'Alex, Jacob and 2 others like this');
     });
   });
   ```


