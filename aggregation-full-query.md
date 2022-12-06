{
      $lookup: {
        from: "paymentrequests",
       // localField: "_id",
        //foreignField: "payLaterPlan.0._id",
        let: { planId: "$_id" },
        pipeline: [
        {
            $unwind: { path: "$payLaterPlan",  preserveNullAndEmptyArrays: true },
        },
        
          {
             
            $match: {
              $expr: {
                  $and: [
                        { $eq: ["$payLaterPlan._id", "$$planId"] },
                         { $gte: ["$createdAt", new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)] },
                         { $in: ["$paymentStatus", ["Info-required", "Pending", "Re-sent", "Submitted", "Cancelled", "Expired", "Active", "Extended", "Completed"]] }
                      ]
            }
        
            },
          },
        ],
        
        as: "paymentRequest"
      }
    },
    {
      $sort: { createdAt: 1 }
    },
    {
      $match: {
        paymentRequest: { $gt: { $size: 0 } },
      }
    },
