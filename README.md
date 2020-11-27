# DB.SQL.CTE

This is solution of Challenge#4.
It mainly has 4 parts. Please not I have created fiddle for each solution. Just click the link to see it running. Please read till end to see the possible recommendations to improve the over all performance.
**1**
[SQLFiddle - click here to run it](http://sqlfiddle.com/#!18/8292a/1)


    with cte_a (Id, ParentId, Hierarchy, HasCircularReference)
    as
    (select Id, ParentId, cast (':'+ convert(nvarchar,Id) as nvarchar) as Hierarchy, 0 as HasCircularReference
    from person
    where ParentId is null
    union all
    select p.Id, p.ParentId, cast(concat_ws(':',cte_a.Hierarchy,p.Id) as nvarchar) as Hierarchy, 
    	CASE  WHEN CHARINDEX(cast (':'+ convert(nvarchar,p.Id) + ':' as nvarchar), cte_a.Hierarchy)>0 THEN 1 ELSE 0 END
    from cte_a inner join person p on cte_a.Id = p.ParentId
    	and cte_a.HasCircularReference = 0
    )
    select * from cte_a
    order by Hierarchy
    OPTION (MAXRECURSION 0);


**2**
[SQLFiddle - click here to run it](http://sqlfiddle.com/#!18/1dfbd/1)


    with cte_a (Id, ParentId, Hierarchy, HasCircularReference)
    as
    (select Id, ParentId, cast (':'+ convert(nvarchar,Id) as nvarchar) as Hierarchy, 0 as HasCircularReference
    from person
    where Id<=2
    union all
    select p.Id, p.ParentId, cast(concat_ws(':',cte_a.Hierarchy,p.Id) as nvarchar) as Hierarchy, 
    	CASE  WHEN CHARINDEX(cast (':'+ convert(nvarchar,p.Id) + ':' as nvarchar), cte_a.Hierarchy)>0 THEN 1 ELSE 0 END
    from cte_a inner join person p on cte_a.Id = p.ParentId
    	and cte_a.HasCircularReference = 0
    )
    select * from cte_a
    order by Hierarchy
    OPTION (MAXRECURSION 0);

**3a**
[SQLFiddle - click here to run it](http://sqlfiddle.com/#!18/1dfbd/3)

    with cte_a (Id, ParentId, Hierarchy, HasCircularReference)
    as
    (select Id, ParentId, cast (':'+ convert(nvarchar,Id) as nvarchar) as Hierarchy, 0 as HasCircularReference
    from person 
    where Id <=2
    union all
    select p.Id, p.ParentId, cast(concat_ws(':',cte_a.Hierarchy,p.Id) as nvarchar) as Hierarchy, 
    	CASE  WHEN CHARINDEX(cast (':'+ convert(nvarchar,p.Id) + ':' as nvarchar), cte_a.Hierarchy)>0 THEN 1 ELSE 0 END
    from cte_a inner join person p on cte_a.Id = p.ParentId
    	and cte_a.HasCircularReference = 0
    )
    ,
    cte_b (Id, ParentId, Hierarchy, HasCircularReference)
    as
    (select Id, ParentId, Hierarchy, HasCircularReference
    from cte_a
    where Id=8
    union all
    select a.Id, a.ParentId, a.Hierarchy, a.HasCircularReference
    from cte_a a inner join cte_b b on a.Id = b.ParentId
    where a.HasCircularReference = 0 and b.ParentId != 8
    )
    select * from cte_b
    where Id != 8
    order by Hierarchy
    OPTION (MAXRECURSION 0)

**3b**
[SQLFiddle - click here to run it](http://sqlfiddle.com/#!18/1dfbd/2)

    with cte_a (Id, ParentId, Hierarchy, HasCircularReference)
    as
    (select Id, ParentId, cast (':'+ convert(nvarchar,Id) as nvarchar) as Hierarchy, 0 as HasCircularReference
    from person
    where Id<=2
    union all
    select p.Id, p.ParentId, cast(concat_ws(':',cte_a.Hierarchy,p.Id) as nvarchar) as Hierarchy, 
    	CASE  WHEN CHARINDEX(cast (':'+ convert(nvarchar,p.Id) + ':' as nvarchar), cte_a.Hierarchy)>0 THEN 1 ELSE 0 END
    from cte_a inner join person p on cte_a.Id = p.ParentId
    	and cte_a.HasCircularReference = 0
    )
    select * from cte_a
    where id != 2 and Hierarchy like ':2:%'
    order by Hierarchy
    OPTION (MAXRECURSION 0);

1. Use of hierarchy data. 
2. Indexing table on parentId (assuming Id is primary key and already indexed)
3. Using graph database for traversing type data.
