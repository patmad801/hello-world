--NAV Prod Order Component Demand - date range
SELECT 
	 poc.[Item No_] AS [Item Number]
	,poc.[Description] AS [Item Desc.]
	,(SELECT CONVERT(INT,COALESCE(SUM(ile.[Remaining Quantity]),0))
			FROM [NAV].[dbo].[BDEL$Item Ledger Entry] ile WITH(NOLOCK)
			WHERE ile.[Item No_] = poc.[Item No_]
				AND ile.[Location Code] IN ('91','90') 
				AND ile.[Open] = 1
				AND ile.[Remaining Quantity] > 0
	   ) 
	   - (CONVERT(INT,COALESCE(SUM(poc.[Remaining Quantity]),0))) --AS [Demand]
		 AS [Shortage]
	
	,po.[Source No_] AS [Parent SKU]
	,po.[Description] AS [Parent Desc.]
	,(SELECT inneri.[Product Sub Group] 
				   FROM [NAV].[dbo].[BDEL$Item] AS inneri WITH(NOLOCK)
				   WHERE poc.[Item No_] = inneri.[No_]
				   ) AS [Category]
	
FROM [NAV].[dbo].[BDEL$Prod_ Order Component] AS poc WITH(NOLOCK)
	JOIN [NAV].[dbo].[BDEL$Production Order] AS po WITH(NOLOCK)
		ON poc.[Prod_ Order No_] = po.[No_]
	JOIN [NAV].[dbo].[BDEL$Item] AS i WITH(NOLOCK)
		ON poc.[Item No_] = i.[No_]

WHERE (poc.[Status] IN(2,3))
	AND (i.[Product Sub Group] IN ('CAMULTRLT','X4','C4'))
	AND (po.[Source No_] LIKE 'BD%')
    AND (poc.[Remaining Quantity] > $0) 
	AND (poc.[Due Date] < GetDate() + 20)
	AND (poc.[Location Code] IN ('90','91'))

GROUP BY 
	 poc.[Item No_]
	,poc.[Description]
	,po.[Source No_]
	,po.[Source No_]
	,po.[Description]

HAVING (SELECT CONVERT(INT,COALESCE(SUM(ile.[Remaining Quantity]),0))
			FROM [NAV].[dbo].[BDEL$Item Ledger Entry] ile WITH(NOLOCK)
			WHERE ile.[Item No_] = poc.[Item No_]
				AND ile.[Location Code] IN ('91','90') 
				AND ile.[Open] = 1
				AND ile.[Remaining Quantity] > 0
		) - (CONVERT(INT,COALESCE(SUM(poc.[Remaining Quantity]),0))) < 0

ORDER BY 
	 [Category] DESC
	,[Item Number] DESC
