---
date: 2016-11-05T09:17:00.000Z
lastmod: 2017-07-30T09:22:27.000Z
title: Ruby and Java Stack Level
draft: false
slug: ruby-and-java-stack-level
tags: ["Ruby","Java","recursion"]

description: 
---

While coding for an algorithmic problem, I discovered that Ruby's stack level is much shallower than Java. This caused a recursive DFS solution written in Ruby failed due to `stack level too deep (SystemStackError)`, while the same code written in Java passed. Whether recursion or tail recursion should be used is not the point of this post. This post is to find out what the max stack level is and what limits the stack level.

## Ruby

Consider the following code

    def recurse(n)
      return 1 if n == 1
      1 + recurse(n-1)
    end
    
    def binary_search
      answer, a, b = 0, 0, 1_000_000_000
    
      while a<=b
        mid = (a+b)/2
        begin
          recurse(mid)
          answer = mid
          a = mid + 1
        rescue SystemStackError
          b = mid - 1
        end
      end
    
      answer
    end
    
    puts "Max Stack Level: #{binary_search}"
    

What do you think `Max Stack Level` is?

Running the code reports that the the `Max Stack Level` is **10080** on my MBP. This is because the stack size of the default Ruby 2.3.1 VM (MRI) is limited to **1MB**.

This is very limiting for even a medium-sized dataset. To increase it, one can [specify VM stack size](http://magazine.rubyist.net/?Ruby200SpecialEn-note#l16) for Ruby >= 2.0.0. To check current max stack size, open `irb`

    irb(main):001:0> RubyVM::DEFAULT_PARAMS
    {
             :thread_vm_stack_size => 1048576,
        :thread_machine_stack_size => 1048576,
              :fiber_vm_stack_size => 131072,
         :fiber_machine_stack_size => 524288
    }
    

Let's set `RUBY_THREAD_VM_STACK_SIZE` to 2MB

    export RUBY_THREAD_VM_STACK_SIZE=2097152
    

And running the same code again returns `Max Stack Level:  20162`, which is roughly 2 * **10080**. Setting `RUBY_THREAD_VM_STACK_SIZE` to 3MB will increase `Max Stack Level` to **30243**. This proves that `Max Stack Level` is linearly proportional to `RUBY_THREAD_VM_STACK_SIZE` in Ruby.

## Java

    public class Solution {
        public static void main(String[] args) {
            System.out.println("Max Stack Level: " + binarySearch());
        }
    
        private static long binarySearch() {
            long a = 0, b = Long.MAX_VALUE, answer = 0;
            while (a <= b) {
                long mid = (a + b) / 2;
                try {
                    recurse(mid);
                    answer = mid;
                    a = mid + 1;
                } catch (StackOverflowError e) {
                    b = mid - 1;
                }
            }
            return answer;
        }
    
        private static long recurse(long n) {
            if (n == 1) return 1;
            return 1 + recurse(n - 1);
        }
    }
    

Running this code with

    java Solution
    

returns `Max Stack Level: 49150`. This is because the thead stack size on my MBP defaults to **1MB**. The defaults are platform-specific, see [here](http://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/jrdocs/refman/optionX.html#wp1024112).

JVM has an option [ss](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/jrdocs/refman/optionX.html#wp999540) for setting the thread stack size. Running the code with thread stack size of 2M

    java -Xss2M Solution
    

returns `Max Stack Level: 98302`, which roughly doubled the stack level. Running it with 4MB gives **196606** and it matches what I thought. Therefore, in Java, `Max Stack Level` is linearly proportional to `ss`.

## Conclusion

Both Ruby and Java have clearly defined max stack size, which limits the number of levels code can recurse. In Ruby, it's defined by environment variable `RUBY_THREAD_VM_STACK_SIZE`. While in Java, it's defined by JVM option `ss`. A clear observation is that when given the same max stack size and the same code, Java performs much better than Ruby. This is not a surprise because Java (on JVM) is a compiled, while Ruby MRI is interpreted.
