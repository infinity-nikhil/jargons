## Locking the db 

### What it means:
It simply means that when lets say user: 23 performing an operation on the db could update or create, another user cannot do the write operation on db untill and unless user 23 doesn't finishes. 
In other words no two users can update or do a write operation on db simuntaniously. 
It is a queue type operation one user at a time 

### Where is can be used:
Well that depends on what use case you are building but for most common use case will be dealing with finance or selling or payment related stuffs 
such that for only one item no 2 user can buy simontaneously (pardon the typo)

Or if you can think more complex then on an order book where orders have to match so an user can get a valid deal and so that no two user can buy and sell according to an price on the orderbook and later one of the two doesn't end up thinking that they got scammed. 

### Enough of general knowledge lets get right into code 
The code is pretty simple some jargon are provided by prisma itself to suppot this architecture 
``` typescript
app.post('/buy', middleware ,async (req: Request, res: Response) => {
    const { marketId } = req.body;
    await prisma.$transaction(async tx => {
        await tx.$queryRaw`SELECT * FROM "Market" WHERE id=${marketId} FOR UPDATE`
        await new Promise(r => setTimeout(r, 4000))
        tx.market.update({
            data: {
                title: "new title"
            },
            where: {
                id: marketId
            }
        })
    })
    res.json({
        message: "Hi"
    })
})
```

Here in promise you can see there is a delay of 4 sec for experiment 

``` typescript
app.get("/market", async(req,res) => {
    const marketId = req.query.marketId as string;
    console.log(marketId);
    const market = await prisma.market.update({
        data: {
            title: "!1234566"
        }, 
        where: {
            id: marketId
        }
    })
    res.json({
        market
    })
} )
```

And meanwhile when you try to update the db you will get a delay 