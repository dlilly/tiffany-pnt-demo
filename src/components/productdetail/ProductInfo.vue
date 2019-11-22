<template>
  <div v-if="product">
    <div data-test="product-gallery"
         class="col-md-4 col-md-offset-1 col-sm-6 product-gallery">
      <ProductGallery :sku="sku" />
    </div>
    <div data-test="product-data"
         class="col-sm-6 product-description">
      <div class="row">
        <div class="col-sm-12">
            <h1 data-test="product-name"
                class="text-uppercase pdp-product-title">
              {{ currentProduct.name }}
            </h1>
            <span data-test="product-sku"
                  class="grey-p quickview-sku">
              {{ sku }}
            </span>
        </div>
      </div>
      <div class="row">
        <div class="col-sm-12">
          <!-- {{> catalog/product-rating }} -->
        </div>
      </div>
      <div class="row">
        <div class="col-sm-12">
          <p class="pdp-product-description view-details more"
             data-text-show="$t('showMore')"
             data-text-hide="$t('showLess')">
          </p>
        </div>
      </div>

      <div class="row">
        <div v-if="hasPrice"
             class="col-sm-12">
          <p class="product-price">
            <BasePrice :price="matchingVariant.price"/>
          </p>

        </div>
      </div>
      <VariantSelector :sku="sku" />
      <AddToCartForm :sku="sku"/>

<!-- Button trigger modal -->
<button type="button" class="btn btn-primary" data-toggle="modal" data-target="#exampleModal">
  Launch demo modal
</button>

<!-- Modal -->
<div class="modal fade" id="exampleModal" tabindex="-1" role="dialog"
  aria-labelledby="exampleModalLabel" aria-hidden="true">
  <div class="modal-dialog modal-lg" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="exampleModalLabel">Modal title</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        <iframe :src="pntURL" width="600"
          height="400" frameborder="0" allowtransparency="true">
        </iframe>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
        <button type="button" class="btn btn-primary">Save changes</button>
      </div>
    </div>
  </div>
</div>

      <!-- {{> catalog/add-to-wishlist-btn}}
      {{> catalog/reserve-in-store-btn}} -->
      <div class="row">
        <div class="col-sm-12">
          <!-- {{> catalog/product-availability availability=product.availability}} -->
        </div>
      </div>
      <DetailsSection :sku="sku"/>
      <SocialMediaLinks/>
    </div>
  </div>
</template>

<script>
import gql from 'graphql-tag';
import productMixin from '@/mixins/productMixin';
import ProductGallery from './ProductGallery.vue';
import SocialMediaLinks from './SocialMediaLinks.vue';
import DetailsSection from './DetailsSection.vue';
import AddToCartForm from './AddToCartForm.vue';
import BasePrice from '../common/BasePrice.vue';
import VariantSelector from './VariantSelector.vue';

export default {
  components: {
    DetailsSection,
    ProductGallery,
    SocialMediaLinks,
    AddToCartForm,
    BasePrice,
    VariantSelector,
  },

  props: {
    sku: {
      type: String,
      required: true,
    },
  },

  data: () => ({
    product: null,
  }),

  computed: {
    matchingVariant() {
      return this.currentProduct.variant || {};
    },
    pntURL() {
      return `https://medusa.mousetrap.tech/tiffany-price-and-time-guide?sku=${this.sku}&locale=${this.$store.state.country}`;
    },
  },

  mixins: [productMixin],

  apollo: {
    product: {
      query: gql`
        query Product($locale: Locale!, $sku: String!, $currency: Currency!) {
          product(sku: $sku) {
            id
            masterData {
              current {
                name(locale: $locale)
                slug(locale: $locale)
                variant(sku: $sku) {
                  price(currency: $currency) {
                    value {
                      ...printPrice
                    }
                    discounted {
                      value {
                       ...printPrice
                      }
                    }
                  }
                }
              }
            }
          }
        }

        fragment printPrice on BaseMoney {
          centAmount
          fractionDigits
        }`,
      variables() {
        return {
          locale: this.$store.state.locale,
          currency: this.$store.state.currency,
          sku: this.sku,
        };
      },
    },
  },

};
</script>

<i18n>
de:
  showMore: "Mehr"
  showLess: "Weniger"
en:
  showMore: "Show more"
  showLess: "Show less"
</i18n>
