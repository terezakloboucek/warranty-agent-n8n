# Scheduled task: "Morning warranty check"

- **Schedule (cron):** `0 9 * * *`
- **Time zone:** your IANA time zone, e.g. `Europe/Prague`

## Task description (paste into the task)

Daily warranty check. Follow these steps exactly:

1. Call `get_settings`. If `telegram_user_id` is empty, stop (send nothing).
2. Call `warranties_due`. If `count` = 0, stop without sending any message.
3. For EACH returned product:
   a) Use web search to find the most frequently reported defects of that specific product (manufacturer + model; search in English and in the language of the receipt: reviews, forums, complaints). Pick 3–5 genuinely recurring problems and give a source (link) for each.
   b) If the manufacturer or model is missing, or nothing relevant is found for the model, list 3–5 typical defects for that product category and say clearly that this is a general overview for this type of product.
   c) Send the owner a direct Telegram message to `telegram_user_id`, in the owner's language, along these lines:
      "⏰ The warranty on *{name}* ({store}, bought {purchase date}) ends on {warranty until}, {days_left} days left (reminder: {reminder}).
      Known defects of this model: • … (source) • …
      Does any of this bother you, or anything else? There is still time to file a claim."
   d) Only after the message was sent successfully, call `mark_notified` with `product_id` and `thresholds` = exactly the `thresholds_to_mark` value of that product (leave status empty).
4. If there are more than 5 products, send detailed messages for the 5 with the fewest days left and one summary message listing the rest (name + warranty until); mark them as well.
5. If sending fails, do not mark the product; it will be retried tomorrow.
