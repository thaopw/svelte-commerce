<script>
// import { storeStore } from '$lib/store/store'
import { browser } from '$app/environment'
import { currency, date } from '$lib/utils'
import { LazyImg } from '$lib/components'
import { onMount } from 'svelte'
import { page } from '$app/stores'
import { PrimaryButton, BackButton } from '$lib/ui'
import dayjs from 'dayjs'
import noAddToCartAnimate from '$lib/assets/no/add-to-cart-animate.svg'
import OrderListSkeleton from '../_OrderListSkeleton.svelte'
import OrderTracking from '../_OrderTracking.svelte'
import productNonVeg from '$lib/assets/product/non-veg.png'
import productVeg from '$lib/assets/product/veg.png'
import ReturnTracking from '../_ReturnTracking.svelte'
import SEO from '$lib/components/SEO/index.svelte'
import TransparentButton from '../_TransparentButton.svelte'

export let data

let clazz
export { clazz as class }

const seoProps = {
	title: 'Order Details ',
	metaDescription: 'Order Details '
}

// let deliveryBy = null
let now = null
let selectedProduct = null
let showDemoScheduler = false
let loading = false

function head() {
	return {
		title: 'Order Details'
	}
}
// let store = {}
$: store = $page.data.store

onMount(() => {
	// if (browser) {
	// 	storeStore.subscribe((value) => (store = value))
	// }
	// deliveryBy = dayjs().add(7, 'day').format('dddd, MMM D, YYYY')
	now = dayjs()
})
</script>

<SEO {...seoProps} />

<div class="{clazz}">
	{#if loading}
		<OrderListSkeleton />
	{:else if data.order}
		<section>
			<BackButton to="/my/orders?sort=-updatedAt" clazz="mb-5" />

			{#each data.order as order}
				<div class="mb-5 overflow-hidden rounded border sm:mb-10">
					<div class="flex flex-wrap items-center justify-between border-b bg-zinc-100 px-5 py-3">
						<h6>Order No : #{order?.orderNo}</h6>

						<h6>Order Date : {date(order?.createdAt)}</h6>
					</div>

					<!-- Order detail  -->

					<div class="grid grid-cols-1 divide-y lg:grid-cols-2 lg:divide-y-0 lg:divide-x">
						<div class="col-span-1 flex flex-col divide-y">
							{#if order?.items}
								<div class="divide-y divide-dashed text-xs text-zinc-500">
									{#each order?.items as item}
										{#if item}
											<div class="flex gap-2 p-5 lg:gap-5">
												<a
													href="{`/product/${item.slug}`}"
													aria-label="Click to view the product details"
													class="shrink-0">
													{#if item.isCustomized}
														<img
															src="{item.customizedImg}"
															alt=" "
															class="h-auto w-14 object-contain object-top" />
													{:else}
														<LazyImg
															src="{item.img}"
															alt=" "
															width="56"
															class="h-auto w-14 object-contain object-top" />
													{/if}
												</a>

												<div class="flex w-full flex-1 flex-col gap-0.5 xl:pr-4">
													<div class="flex justify-between gap-2 sm:gap-4">
														<a
															href="{`/product/${item.slug}`}"
															aria-label="Click to view the product details"
															class="flex-1 hover:underline">
															<p>
																{item.name}
															</p>
														</a>

														{#if $page.data.store?.isFnb && item.foodType}
															<div>
																{#if item.foodType === 'veg'}
																	<img src="{productVeg}" alt="veg" class="h-5 w-5" />
																{:else if item.foodType === 'nonveg'}
																	<img src="{productNonVeg}" alt="non veg" class="h-5 w-5" />
																{/if}
															</div>
														{/if}
													</div>

													{#if item.qty}
														<span>
															Qty :

															<b>
																{item.qty}
															</b>
														</span>
													{/if}

													{#if item.size}
														<span>
															Size :

															<b>
																{item.size}
															</b>
														</span>
													{/if}

													{#if item?.usedOptions?.length}
														{#each item?.usedOptions as option}
															{#if option?.val?.length && option?.val !== undefined && option?.val != ''}
																<div class="flex flex-wrap gap-2">
																	<span>{option.name}:</span>

																	{#if option.val}
																		<ul class="flex flex-wrap items-center gap-x-2 gap-y-1">
																			{#each option.val as v, valIndex}
																				{#if v}
																					<b>
																						{v}
																					</b>

																					{#if valIndex < option.val?.length - 1}
																						,
																					{/if}
																				{/if}
																			{/each}
																		</ul>
																	{/if}
																</div>
															{/if}
														{/each}
													{/if}

													<div class="flex flex-wrap items-center gap-1">
														Item price :

														<span class="font-bold whitespace-nowrap text-zinc-800">
															{currency(item.price, store?.currencySymbol)}
														</span>

														{#if item?.mrp > item?.price}
															<span class="whitespace-nowrap text-zinc-500 line-through">
																<strike>
																	{currency(item.mrp, store?.currencySymbol)}
																</strike>
															</span>

															{#if Math.floor(((item.mrp - item.price) / item.mrp) * 100) > 0}
																<span class="whitespace-nowrap text-secondary-500">
																	({Math.floor(((item.mrp - item.price) / item.mrp) * 100)}% off)
																</span>
															{/if}
														{/if}
													</div>

													<div class="flex flex-wrap items-center gap-1">
														Sub Total :

														<span class="font-bold whitespace-nowrap text-zinc-800">
															{currency(item.subtotal, store?.currencySymbol)}
														</span>

														{#if item?.total > item?.subtotal}
															<span class="whitespace-nowrap text-zinc-500 line-through">
																<strike>
																	{currency(item.total, store?.currencySymbol)}
																</strike>
															</span>

															{#if Math.floor(((item.total - item.subtotal) / item.total) * 100) > 0}
																<span class="whitespace-nowrap text-secondary-500">
																	({Math.floor(((item.total - item.subtotal) / item.total) * 100)}%
																	off)
																</span>
															{/if}
														{/if}
													</div>

													{#if item?.files?.length}
														<ul class="mt-2 p-0 list-none flex flex-col gap-1">
															{#each item?.files as file, fx}
																<li>
																	<a href="{file}" download>
																		<PrimaryButton loadingringsize="xs" class="text-xs">
																			Download File

																			{#if item?.files?.length > 1}
																				{fx + 1}
																			{/if}
																		</PrimaryButton>
																	</a>
																</li>
															{/each}
														</ul>
													{/if}

													{#if item?.status === 'delivered'}
														<div class="mt-2 xl:mt-0 xl:w-1/3">
															<a
																href="/my/reviews/create?pid={item?.pid}&oid={item?.orderItemId}&ref=/product/{item?.slug}"
																aria-label="Click to visit rate & review product"
																class="max-w-max whitespace-nowrap font-semibold text-indigo-500 focus:outline-none hover:underline">
																Rate & Review Product
															</a>
														</div>
													{/if}
												</div>
											</div>
										{/if}
									{/each}
								</div>

								<div class="p-5 flex justify-end">
									<div class="flex max-w-max flex-col items-start gap-2">
										<p class="flex items-center">
											<span class="mr-2 w-32">Subtotal</span>

											<span>
												: &nbsp; {currency(order?.amount?.subtotal, store?.currencySymbol)}
											</span>
										</p>

										<p class="flex items-center">
											<span class="mr-2 w-32">Discount</span>

											<span>
												: &nbsp;

												{currency(order?.amount?.discount, store?.currencySymbol)}
											</span>
										</p>

										{#if order?.coupon?.code}
											<p class="flex items-center">
												<span class="mr-2 w-32">Applied Coupon</span>

												<span>
													: &nbsp;
													{order?.coupon?.code}
												</span>
											</p>
										{/if}

										<p class="flex items-center">
											<span class="mr-2 w-32">Shipping</span>

											<span>
												: &nbsp;
												{#if order?.amount?.shipping}
													{currency(order?.amount?.shipping, store?.currencySymbol)}
												{:else}
													Free
												{/if}
											</span>
										</p>

										{#if order?.amount?.cod_charges}
											<p class="flex items-center">
												<span class="mr-2 w-32">COD Charges</span>

												<span>
													: &nbsp;

													{currency(order?.amount?.cod_charges, store?.currencySymbol)}
												</span>
											</p>
										{/if}

										<hr class="w-full border-t border-zinc-200" />

										<div class="flex items-center text-sm font-bold text-zinc-800">
											<span class="mr-2 w-32">Total</span>

											<span>
												: &nbsp; {currency(order?.amount?.total, store?.currencySymbol)}
											</span>
										</div>
									</div>
								</div>
							{:else}
								<p>No order items found</p>
							{/if}
						</div>

						<div class="col-span-1 flex flex-col gap-5 p-5 lg:gap-10">
							<div>
								<h5 class="mb-2">Delivery Address</h5>

								<p class="flex flex-col">
									<span>
										{order?.shipping_address?.firstName}
										{order?.shipping_address?.lastName}

										<br />

										{order?.shipping_address?.address}, {order?.shipping_address?.city},
										{order?.shipping_address?.country}, {order?.shipping_address?.state}

										<br />

										{order?.shipping_address?.zip}
									</span>
								</p>

								{#if order?.shipping_address?.phone}
									<p>
										{order?.shipping_address?.phone}
									</p>
								{/if}
							</div>

							<div>
								<h5 class="mb-2">Billing Address</h5>

								<p class="flex flex-col">
									<span>
										{order?.billing_address?.firstName}
										{order?.billing_address?.lastName}

										<br />

										{order?.billing_address?.address}, {order?.billing_address?.city},
										{order?.billing_address?.country}, {order?.billing_address?.state}

										<br />

										{order?.billing_address?.zip}
									</span>
								</p>

								{#if order?.billing_address?.phone}
									<p>
										{order?.billing_address?.phone}
									</p>
								{/if}
							</div>

							<!-- <div>
							<h5 class="mb-2">Vendor Details :</h5>

							<p class="flex flex-col">
								<span>
									{order?.vendorBusinessName},

									{order?.vendorAddress?.address}, {order?.vendorAddress?.town},

									{order?.vendorAddress?.city}, {order?.vendorAddress?.state}</span>

								<span>{order?.vendorAddress?.zip}</span>
							</p>

							{#if order?.vendorPhone}
								<h6 class="mt-2">
									Phone number: <span> {order?.vendorPhone}</span>
								</h6>
							{/if}
						</div> -->
						</div>
					</div>
				</div>
			{/each}
			<!-- Order Tracker -->

			<div>
				{#if !!data.order?.foodType && data.order?.status !== 'delivered' && data.order?.expectedDeliveryDate}
					<div class="mb-5 flex items-center gap-2 flex-wrap">
						<h5>Expected Delivery Date :</h5>

						<p>
							{date(data.order?.expectedDeliveryDate)}
						</p>
					</div>
				{/if}

				{#if data.order.tracking_link}
					<a
						href="{data.order.tracking_link}"
						target="_blank"
						rel="noopener noreferrer"
						class="mb-5 font-semibold text-indigo-500 underline hover:text-indigo-700">
						Track your order
					</a>
				{/if}

				<div class="mt-5 sm:mt-10 flex flex-wrap gap-10">
					{#if data.orderTracking?.length}
						<OrderTracking tracks="{data.orderTracking}" />
					{/if}

					<!-- {#if !data.order?.isReplaceOrReturn}
					{:else}
						<ReturnTracking order="{data.order}" />
					{/if} -->

					<div class="flex flex-col gap-2">
						{#if data.order?.invoiceLink}
							<a
								href="{data.order?.invoiceLink}"
								aria-label="Click to download invoice"
								target="blank"
								class="block">
								<PrimaryButton class="w-48" type="button">Download Invoice</PrimaryButton>
							</a>
						{/if}

						{#if data.order?.replaceValidTill != null && now <= data.order?.replaceValidTill && !data.order?.isReplaceOrReturn}
							<a
								href="/my/exchange?orderId=${data.order?.orderId}&itemId=${data.order?.itemId}"
								aria-label="Click to visit exchange"
								class="block">
								<TransparentButton class="w-48" type="button" border>Exchange</TransparentButton>
							</a>
						{/if}

						<!-- {#if data.order?.returnValidTill != null && now <= data.order?.returnValidTill && !data.order?.isReplaceOrReturn}
							<a
								href="/my/return?orderId=${data.order?.orderId}&itemId=${data.order?.itemId}"
								aria-label="Click to visit return"
								class="block">
								<TransparentButton class="w-48" type="button" border>Return</TransparentButton>
							</a>
						{/if} -->
					</div>
				</div>
			</div>
		</section>
	{:else}
		<div class="flex h-[70vh] flex-col items-center justify-center text-center">
			<img src="{noAddToCartAnimate}" alt="empty wishlist" class="mb-5 h-60 object-contain" />

			<h2 class="mb-2">Your have't Ordered Yet !!</h2>

			<p class="mb-5">Add items to it now</p>

			<a href="/" aria-label="Click to visit home" data-sveltekit-preload-data>
				<PrimaryButton class="w-40 py-2 text-sm">Shop Now</PrimaryButton>
			</a>
		</div>
	{/if}
</div>
