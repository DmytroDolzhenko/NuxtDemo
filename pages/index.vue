<script setup lang="ts">
import { ref, computed } from 'vue'
import { useFetch } from '#app'
import { h, resolveComponent } from 'vue'
import type { TableColumn } from '@nuxt/ui'
import type { Column } from '@tanstack/vue-table'
import  { getPaginationRowModel} from '@tanstack/vue-table'
const UAvatar = resolveComponent('UAvatar')

const UBadge = resolveComponent('UBadge')
const UButton = resolveComponent('UButton')
const UDropdownMenu = resolveComponent('UDropdownMenu')

const table = useTemplateRef('table')

type Product = {
  id : number
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: { src: string, alt: string }
}

type ProductResponse = {
  products: Product[]
}

const { data, status } = await useFetch<Product[]>('https://dummyjson.com/products', {
  key: 'table-products',
  transform: (response: ProductResponse): Product[] => {
    return response.products.map(product => ({
      ...product,
      thumbnail: {
        src: product.thumbnail,
        alt: `${product.title} thumbnail`
      },
    }))
  },
  lazy: true
})


const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'title',
    header: ({ column }) => getHeader(column, 'Title'),
    cell: ({ row }) => {
      return h('div', { class: 'flex items-center gap-3' }, [
        h(UAvatar, {
          src: row.original.thumbnail.src,
          alt: row.original.thumbnail.alt,
          class: 'my-avatar'
        }),
        h('div', undefined, [
          h('p', { class: 'font-medium text-(--ui-text-highlighted)' }, row.original.title),
        ])
      ])
    }
  },
  {
    accessorKey: 'description',
    header: ({ column }) => getHeader(column, 'Description'),
    cell: ({ row }) => row.original.description,
  },
  {
    accessorKey: 'price',
    header: ({ column }) => getHeader(column, 'Price'),
    cell: ({ row }) => row.original.price
  },
  {
    accessorKey: 'rating',
    header: ({ column }) => getHeader(column, 'Rating'),
    cell: ({ row }) => {
      const rating = row.original.rating
      const ratingColor = rating > 4.5 ? 'text-green-500' : 'text-red-500'
      return h('span', { class: `font-semibold ${ratingColor}` }, rating)
    }
  },
  {
    accessorKey: 'brand',
    header: ({ column }) => getHeader(column, 'Brand'),
    cell: ({ row }) => row.original.brand
  },
  {
    accessorKey: 'category',
    header: ({ column }) => getHeader(column, 'Category'),
    cell: ({ row }) => row.original.category
  }
]

function getHeader(column: Column<Product>, label: string) {
  const isSorted = column.getIsSorted()

  return h(
      UDropdownMenu,
      {
        content: {
          align: 'start'
        },
        'aria-label': 'Actions dropdown',
        items: [
          {
            label: 'Asc',
            type: 'checkbox',
            icon: 'i-lucide-arrow-up-narrow-wide',
            checked: isSorted === 'asc',
            onSelect: () => {
              if (isSorted === 'asc') {
                column.clearSorting()
              } else {
                column.toggleSorting(false)
              }
            }
          },
          {
            label: 'Desc',
            icon: 'i-lucide-arrow-down-wide-narrow',
            type: 'checkbox',
            checked: isSorted === 'desc',
            onSelect: () => {
              if (isSorted === 'desc') {
                column.clearSorting()
              } else {
                column.toggleSorting(true)
              }
            }
          }
        ]
      },
      () =>
          h(UButton, {
            color: 'neutral',
            variant: 'ghost',
            label,
            icon: isSorted
                ? isSorted === 'asc'
                    ? 'i-lucide-arrow-up-narrow-wide'
                    : 'i-lucide-arrow-down-wide-narrow'
                : 'i-lucide-arrow-up-down',
            class: '-mx-2.5 data-[state=open]:bg-(--ui-bg-elevated)',
            'aria-label': `Sort by ${isSorted === 'asc' ? 'descending' : 'ascending'}`
          })
  )
}

const sorting = ref([
  {
    id: 'id',
    desc: false
  }
])

const pagination = ref({
  pageIndex: 0,
  pageSize: 5
})

const globalFilter = ref('45')


useHead({
  title: 'Список продуктів'
})
</script>
<template>
  <div class="flex flex-col flex-1 w-full">
    <div class="flex px-4 py-3.5 border-b border-(--ui-border-accented)">
      <UInput v-model="globalFilter" class="max-w-sm" placeholder="Filter..." />
    </div>
  </div>
  <div class="w-full space-y-4 pb-4">

    <UTable
        ref="table"
        v-model:pagination="pagination"
        v-model:global-filter="globalFilter"
        :data="data"
        :columns="columns"
        :pagination-options="{
        getPaginationRowModel: getPaginationRowModel()
      }"
        class="flex-1"
    />

    <div class="flex justify-center border-t border-(--ui-border) pt-4">
      <UPagination
          :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
          :items-per-page="table?.tableApi?.getState().pagination.pageSize"
          :total="table?.tableApi?.getFilteredRowModel().rows.length"
          @update:page="(p) => table?.tableApi?.setPageIndex(p - 1)"
      />
    </div>
  </div>
</template>

