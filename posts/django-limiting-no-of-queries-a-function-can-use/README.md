# Django: Limiting no of DB queries a function can execute

Too many DB queries slow down the APIs and affect DB performance. 

Even if an existing function makes only necessary no of queries, future 
developers might add code that increases it unnecessarily.

To flag it for their review during development so that they can see and decide
whether it can be optimized or it is necessary, I wrote a decorator.

It can be used with functions which tend to get new DB queries a lot.

Below is the code:

````python
from django.conf import settings
from functools import wraps

def count_db_queries(func=None, *, max_queries=None):
    """
    Simple decorator to count database queries executed by a function.
    Prints a single line with the query count.


    Args:
        max_queries (int, optional): Maximum allowed queries. Raises exception if exceeded.
        Use this for functions that tend to use too many queries.
    """

    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            if not settings.DEBUG:
                result = func(*args, **kwargs)
                return result
            
            from django.db import connection, reset_queries

            reset_queries()
            initial_queries = len(connection.queries)

            result = func(*args, **kwargs)

            final_queries = len(connection.queries)
            total_queries_executed = final_queries - initial_queries

            # ANSI color codes: green text, bright red for count, reset
            print(
                f"\033[32m{func.__name__} executed \033[91m{total_queries_executed}\033[32m database queries\033[0m"
            )

            # Check if queries exceed max limit
            if max_queries is not None and total_queries_executed > max_queries:
                raise Exception(
                    f"{func.__name__} executed {total_queries_executed} queries, "
                    f"which exceeds the maximum allowed limit of {max_queries}"
                )

            return result

        return wrapper

    # Handle both @count_db_queries and @count_db_queries(max_queries=5)
    if func is None:
        return decorator
    else:
        return decorator(func)

@count_db_queries(max_queries=1)
def a_func_which_cant_use_more_than_one_query():
    pass


````

When a function is flagged for review, the developer can choose either to
review and optimize or just increase the max_queries if it is deemed
necessary. 

Keep in mind that `transaction` is also a query
so that should be considered while counting queries.
