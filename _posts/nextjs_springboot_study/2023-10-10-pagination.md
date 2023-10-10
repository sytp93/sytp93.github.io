---
layout: post
title: next.js and Spring-boot (5)
subtitle: 페이징 기능 적용
cover-img: ./assets/img/path.jpg
thumbnail-img: ./assets/img/thumb.png
share-img: ./assets/img/path.jpg
tags: [project]
---
## Pagination

```javascript
    case "GET":
      const chargerList = await axios.get(apiUrl, {
        params: {
          productCode: req.query.productCode,
          searchType: req.query.searchType,
          searchText: req.query.searchText,
        }
      });
      return res.status(200).json({
        code: 0,
        message: "OK",
        data: {
          items: chargerList.data,
          page: {
            currentPage: page,
            pageSize: 5,
            totalPage: 1,
            totalCount: chargerList.data.length,
          }
        }
      });
```

```javascript
  const handleChangePage = useCallback(
    (pageNumber: number) => {
      router.push({
        pathname: router.pathname,
        query: { ...router.query, page: pageNumber },
      });
    },
    [router]
  );

<DefaultTable<ICharger>
        rowSelection={rowSelection}
        columns={columns}
        dataSource={data?.data.items || []}
        loading={isLoading}
        pagination={{
          current: Number(router.query.page || 1),
          defaultPageSize: 5,
          total: data?.data.page.totalCount || 0,
          showSizeChanger: false,
          onChange: handleChangePage,
        }}
        className="mt-3"
        countLabel={data?.data.page.totalCount}
      />
```

현재 client에서 pagination 설정이 되어있는 코드이다.