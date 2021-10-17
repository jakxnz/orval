---
id: react-query
title: React query
---

You should have an OpenApi specification and an Orval config where you define the mode as react-query.

#### Example with React query

```js
module.exports = {
  petstore: {
    output: {
      mode: 'tags-split',
      target: 'src/petstore.ts',
      schemas: 'src/model',
      client: 'react-query',
      mock: true,
    },
    input: {
      target: './petstore.yaml',
    },
  },
};
```

Checkout the [orval config](../reference/configuration/full-example) reference to see all available options.

The React query model will generate an implementation file with one custom hook per path in your OpenApi Specification.

Like the following example from this <a href="https://github.com/anymaniax/orval/blob/master/samples/react-app-with-react-query/petstore.yaml" target="_blank">swagger</a>:

```ts
export const showPetById = (
  petId: string,
  options?: AxiosRequestConfig,
): Promise<AxiosResponse<Pet>> => {
  return axios.get(`/pets/${petId}`, options);
};

export const getShowPetByIdQueryKey = (petId: string) => [`/pets/${petId}`];

export const useShowPetById = <
  TData = AsyncReturnType<typeof showPetById>,
  TError = Error,
>(
  petId: string,
  options?: {
    query?: UseQueryOptions<AsyncReturnType<typeof showPetById>, TError, TData>;
    axios?: AxiosRequestConfig;
  },
) => {
  const { query: queryOptions, axios: axiosOptions } = options || {};

  const queryKey = queryOptions?.queryKey ?? getShowPetByIdQueryKey(petId);
  const queryFn = () => showPetById(petId, axiosOptions);

  const query = useQuery<AsyncReturnType<typeof queryFn>, TError, TData>(
    queryKey,
    queryFn,
    { enabled: !!petId, ...queryOptions },
  );

  return {
    queryKey,
    ...query,
  };
};
```

### How use other query settings and options

Orval can modify query options for all paths/operations or a subset of paths/operations.

### Query settings and options for all paths/operations

With the following example Orval will generate a useQuery and useInfinteQuery with a nextId queryparam. You can also override the config for each one with the options props.

```js
module.exports = {
  petstore: {
    output: {
      ...
      override: {
        query: {
          useQuery: true,
          useInfinite: true,
          useInfiniteQueryParam: 'nextId',
          options: {
            staleTime: 10000,
          },
        },
      },
    },
    ...
  },
};
```

### Query settings and options for specific paths/operations

If needed you can override specific paths/operations directly via operations or tags

```js
module.exports = {
  petstore: {
    output: {
      ...
      override: {
        operations: {
          listPets: {
            query: {
              ...
            },
          }
        },
        tags: {
          pet: { // Must match OpenApi document
            query: {
              ...
            }
          }
        }
      },
    }
    ...
  },
};
```

Checkout <a href="https://github.com/anymaniax/orval/blob/master/samples/react-app-with-react-query" target="_blank">here</a> the full example
