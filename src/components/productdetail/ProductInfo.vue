<template>
  <div v-if="product">
    <div class="hidden" id="skuHolder" :data-value="sku"></div>
    <div class="hidden" id="cartIdHolder" :data-value="me.activeCart.id"></div>>
    <div class="hidden" id="countryHolder" :data-value="$store.state.country"></div>>

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
<!-- <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#exampleModal"> -->
<button type="button" class="btn btn-primary" data-toggle="modal" id='serviceOptions'>
  Service Options
</button>

<!-- Modal -->
<div class="modal fade" id="exampleModal" tabindex="-1" role="dialog"
  aria-labelledby="exampleModalLabel" aria-hidden="true">
  <div class="modal-dialog modal-lg" role="document">
    <div class="modal-content">
      <div class="modal-body">
        <!-- <iframe id="pntFrame" :src="pntURL" width="600" -->
        <iframe id="pntFrame" width="600"
          height="400" frameborder="0" allowtransparency="true">
        </iframe>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
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
import cartMixin from '@/mixins/cartMixin';
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
    me: null,
    product: null,
  }),

  computed: {
    matchingVariant() {
      return this.currentProduct.variant || {};
    },
  },

  mixins: [productMixin, cartMixin],

  apollo: {
    me: {
      query: gql`
        query me {
          me {
            activeCart {
              id
            }
          }
        }
      `,
    },
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

// const domain = 'https://medusa.mousetrap.tech';
const domain = 'http://localhost:3001';

$(document).on('click', '#serviceOptions', () => {
  const sku = $('#skuHolder').data('value');
  const cartId = $('#cartIdHolder').data('value');
  const country = $('#countryHolder').data('value');

  const frameURL = `${domain}/tiffany-price-and-time-guide?sku=${sku}&locale=${country}&cartId=${cartId}`;
  $('#pntFrame').prop('src', frameURL);
  $('#exampleModal').modal('show');
});

$(document).on('show.bs.modal', '#exampleModal', () => {
  const receiver = document.getElementById('pntFrame').contentWindow;
  setTimeout(() => { receiver.postMessage('addToCart', domain); }, 1000);
});

window.addEventListener('message', (event) => {
  console.log(`message ${JSON.stringify(event.data)}`);
  if (event.data.cart) {
    $('#exampleModal').modal('hide');
    window.location.reload();
  }
}, false);
</script>

<i18n>
de:
  showMore: "Mehr"
  showLess: "Weniger"
en:
  showMore: "Show more"
  showLess: "Show less"
</i18n>
